---
title: "When TLS 1.3 Silently Dies Inside Your Android Proxy"
date: 2026-03-20
draft: false
tags: ["security", "network", "linux", "android", "tls", "go"]
categories: ["Security"]
summary: "A post-mortem of intermittent HTTPS failures across a mobile proxy fleet: TLS 1.3 handshakes silently dying on memory-starved Android devices — large multi-packet handshake messages, inflated by post-quantum key shares, stressing proxy buffers under memory pressure."
canonicalURL: "https://dev.to/rivik/when-tls-13-silently-dies-inside-your-android-proxy-1c3"
ShowCanonicalLink: true
---

We run [iProxy.online](https://iproxy.online), a mobile proxy infrastructure. Our Android app turns phones into proxy servers across 100+ countries. Last year we shipped an advanced network health checker that runs a lot of probes through these proxies to a controlled server. That’s when things got weird.

A small but noticeable percentage of devices started failing HTTPS checks. HTTP worked fine. The failure was always at the TLS handshake stage. And the behavior was completely non-deterministic: broken for two hours, then fine, then broken for five minutes, then fine again.

## What we saw

The correlations were weak and noisy. Android v8-v9 devices showed up more often. Cheaper, lower-spec phones were overrepresented. The strongest signal was memory pressure on the device. When we could catch the failure in real time, the device was almost always low on available RAM. But metrics from memory-starved phones are unreliable by definition, so we couldn’t be sure this wasn’t survivorship bias.

Two tracks of investigation: find correlations (inconclusive, as described above) and understand what actually breaks.

The second track was harder than it sounds. These are not our devices. They sit in remote locations. Physical access happens maybe once a month. We can’t deploy debug builds on demand. The bug is intermittent with no predictable trigger. Direct on-device debugging was effectively impossible.

## Narrowing it down

Our metrics pointed at TLS handshake failure, so we tried to reproduce manually. curl through the same proxy, same server (Caddy, default Ubuntu 24.04 repos):

```shell
curl -x socks5://proxy:port https://our-server.example.com
```

Works perfectly. TLS 1.3, clean handshake, 200 OK. Every time.

Our network checker is written in Go (1.24 at the time). I built a minimal Go client to isolate the behavior. Here’s where it got interesting:

|Client|TLS version |Result|
|------|------------|------|
|curl  |1.3         |works |
|Go    |1.3         |hangs |
|Go    |1.2 (forced)|works |

Forcing TLS 1.2 in Go:

```go
tlsConfig := &tls.Config{
    MaxVersion: tls.VersionTLS12,
}
```

This consistently fixed the issue on affected devices.

A tcpdump on the client side showed the Go client sending ClientHello and then… nothing. No ServerHello coming back to client. The proxy app just sat there.

## It gets weirder

The problem was server-dependent. Go + TLS 1.3 failed against our Caddy server, against Cloudflare, against Google. But it worked against some other sites. So the failure depended on the specific TLS implementation on the remote end, not just the client.

And this wasn’t limited to our checker. When the bug was active on a device, Chrome via same mobile proxy couldn’t open Google over TLS 1.3 either. TLS 1.2 sites loaded fine. This was a device/app-level issue, not something specific to our Go code.

## Why Go and curl behave differently

This is the part that took the longest to reason about.

Go’s `crypto/tls` and curl’s underlying OpenSSL/BoringSSL produce different ClientHello messages. Go’s ClientHello has always been somewhat larger due to different extension sets and key share choices. But there’s a much bigger factor here that we didn’t initially consider.

**Starting with Go 1.23, the post-quantum key exchange X25519Kyber768Draft00 is enabled by default** when `Config.CurvePreferences` is nil (which is the standard case). In Go 1.24, this became X25519MLKEM768. The ML-KEM public key alone is 1184 bytes. This makes the ClientHello big enough to exceed a single TCP packet at typical 1500-byte MTU.

curl (as of the versions we tested) does not send post-quantum key shares by default. Its ClientHello is much smaller and fits comfortably in a single packet.

This size difference matters because our Android app is acting as a TCP proxy. It reads data from one socket and writes it to another. The proxy doesn’t terminate TLS; it just forwards bytes. But it still needs to buffer them.

## The (probable) mechanism

Here’s our working theory. There are two chokepoints, not one.

Chokepoint 1: the ClientHello. Go 1.24 sends a very big ClientHello due to the ML-KEM post-quantum key share (1184 bytes for the public key alone). curl’s OpenSSL sends ~500-700 bytes with a 32-byte X25519 key share. Chrome 124+ also sends post-quantum key shares by default, producing similarly large ClientHello messages. Our tcpdump on the client side showed the ClientHello leaving the client, then silence. This means the proxy either failed to forward the ClientHello to the server, or forwarded it and failed to relay the server’s response back. We can’t distinguish these two cases from a client-side capture alone, but both point to the same thing: the proxy is the bottleneck.

Chokepoint 2: the server flight. This explains why TLS 1.3 worked with some servers but not others, even when the ClientHello is identical.

In TLS 1.3 the server responds with a single flight: ServerHello + EncryptedExtensions + Certificate + CertificateVerify + Finished, all at once. The size of this response depends on the server’s certificate chain. A small site with a single cert and short chain might send 2-3 KB. Google or Cloudflare with full certificate chains send 4-6+ KB. Unfortunately, I don't remember what kind of sites worked with TLS 1.3 and what kind of certificate chain they had, so this is just a guess.

So even when the ClientHello makes it through the proxy, the server response might not. A memory-starved proxy can relay a 2 KB response but chokes on a 5 KB one. That’s why some sites worked and others didn’t, with the exact same client? It sounds dubious, the numbers are too small, but I didn't have any other hypotheses.

TLS 1.2 avoids both problems. Its ClientHello is smaller (no PQ key shares), and the handshake is split across multiple round trips with smaller messages in each direction. No single message is large enough to stress the proxy’s buffers.

Why restart fixes it: killing the app clears leaked memory, resets all socket state, and gives the proxy fresh buffer capacity. The fact that this consistently works is the strongest evidence that the root cause is resource exhaustion, not a protocol bug.​​​​​​​​​​​​​​​​

## The fix

We needed a production fix, not a research paper. We did two things:

**Immediate mitigation**: capped TLS to 1.2 for the checker and for proxy traffic where possible.

```go
tlsConfig := &tls.Config{
    MaxVersion: tls.VersionTLS12,
}
```

**Observability-driven restart**: the checker now detects when TLS 1.3 fails but TLS 1.2 succeeds on the same device. When this pattern appears, we send a remote command to fully restart the app (kill the process, clear memory, relaunch). This consistently fixes the problem, which further supports the memory pressure hypothesis.

We also found and fixed memory leaks in our app that were contributing to the pressure. The correlation between leak fixes and reduced TLS 1.3 failures was visible in our dashboards.

## Why we didn’t dig deeper

We don’t have a verified root cause. We have a strong hypothesis, consistent correlations, and effective mitigations that confirm the hypothesis indirectly.

The devices are remote, not ours, and physically accessible about once a month. The bug is intermittent with no reliable trigger. Production users were affected. We needed fast fix, not a deep research.

We’re now investing in better telemetry and the ability to capture targeted diagnostics remotely. Next time something like this happens, we’ll have the data to pin it down.​​​​​​​​​​​​​​​​

## Key takeaways

**For proxy developers on constrained devices**: TLS 1.3 messages are larger than TLS 1.2, and post-quantum key exchange makes them much larger. If your proxy buffers TCP data, make sure your buffers can handle multi-packet TLS records, especially under memory pressure.

**For Go developers proxying TLS**: be aware that Go 1.23+ sends post-quantum key shares by default. If you’re running through proxies or middleboxes, this can break things. Set `CurvePreferences` explicitly or use `GODEBUG=tlskyber=0` (Go 1.23) / `GODEBUG=tlsmlkem=0` (Go 1.24+) to disable it.

**For anyone debugging intermittent TLS failures**: if TLS 1.2 works and TLS 1.3 doesn’t, the problem is almost certainly not in TLS itself. It’s in something between client and server that can’t handle the larger messages. Check your middleboxes, proxies, and buffer sizes.