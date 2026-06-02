---
title: "Hardware-backed SSH keys end to end: YubiKey, PIV, software alternatives, and where SSH CAs fit in"
date: 2026-05-09
draft: false
tags: ["ssh", "yubikey", "fido2", "piv", "security", "ssh-ca", "linux", "ansible"]
categories: ["Security"]
summary: "A working guide to using a YubiKey for SSH on a real Linux fleet — the four knobs (resident, touch, PIN, agent), a four-mode policy for root and Ansible, software-only alternatives, and where SSH CAs fit in."
canonicalURL: "https://dev.to/rivik/hardware-backed-ssh-keys-end-to-end-yubikey-piv-software-alternatives-and-where-ssh-cas-fit-in-3lob"
ShowCanonicalLink: true
---

This is a working guide to using a YubiKey for SSH on a real Linux fleet, plus the surrounding landscape — PIV, software-only alternatives, and SSH certificate authorities. The goal is to retire file-based SSH keys without breaking daily operations.

The article is structured around four questions:

1. What does a hardware-backed key actually do, and what knobs do you control?
2. How do you combine those knobs into a policy that works for both root login and Ansible?
3. What if you can't ship YubiKeys?
4. When should you stop managing keys yourself and adopt an SSH CA?

---

## The problem with file-based keys

Every classic SSH key is a file in `~/.ssh/`. That file holds the private key. To log in to a server, your SSH client reads the file and produces a cryptographic signature.

There are really two issues here, and they compound:

1. **The private key is a file.** It exists in the filesystem, can be read by anything with sufficient access, can be copied, backed up, accidentally committed, or extracted via a misconfigured recovery scenario. This is a fundamental property of where the key lives.
2. **The discipline that would mitigate this rarely survives daily work.** The cryptography is fine; the operational reality isn't.
   - **What works in theory:** a passphrase-protected key combined with `ssh-agent -t 10m` is genuinely close to unbreakable. The key is decrypted briefly, signs what it needs, and the agent forgets it.
   - **What happens in practice:** engineers drop passphrases for convenience, or load the key into ssh-agent on first use and leave the agent running for the entire session.
   - **Agent forwarding compounds it:** with `ssh -A`, a key that's been unlocked once can sign on the operator's behalf from any forwarded host for the rest of the agent's lifetime.

Hardware-backed keys remove the need for that discipline. The private key never leaves the device, and signing requires the device's physical presence — there's nothing to forget to passphrase, nothing to leave running for too long, nothing for a forwarded host to sign with silently.

YubiKey is the most flexible option because the same device works on Linux, macOS, Windows, iOS, and Android with the same protocol and the same key files. Most of this article is about YubiKey + FIDO2; the alternatives come later.

---

## How a YubiKey actually signs

<svg viewBox="0 0 800 240" xmlns="http://www.w3.org/2000/svg" style="max-width:100%;height:auto">
  <defs>
    <marker id="a1" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="8" markerHeight="8" orient="auto">
      <path d="M0 0 L10 5 L0 10 z" fill="currentColor"/>
    </marker>
  </defs>
  <rect x="40" y="60" width="240" height="130" rx="8" stroke="currentColor" stroke-width="2" fill="none"/>
  <text x="160" y="95" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="20" font-weight="600">Laptop</text>
  <text x="160" y="125" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13" opacity="0.65">SSH client asks the device:</text>
  <text x="160" y="150" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="14" font-style="italic">"sign this nonce"</text>
  <text x="160" y="175" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.55">no private key on disk</text>
  <path d="M285 105 L515 105" stroke="currentColor" stroke-width="2" fill="none" marker-end="url(#a1)"/>
  <text x="400" y="95" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12">request</text>
  <path d="M515 145 L285 145" stroke="currentColor" stroke-width="2" fill="none" marker-end="url(#a1)"/>
  <text x="400" y="165" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12">signature</text>
  <rect x="520" y="60" width="240" height="130" rx="8" stroke="#d4a017" stroke-width="2" fill="none"/>
  <text x="640" y="95" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="20" font-weight="600">YubiKey</text>
  <rect x="620" y="115" width="40" height="32" rx="3" stroke="#d4a017" stroke-width="2" fill="none"/>
  <path d="M628 115 v-10 a12 12 0 0 1 24 0 v10" stroke="#d4a017" stroke-width="2" fill="none"/>
  <text x="640" y="175" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">private key never leaves</text>
</svg>

The SSH client never sees the private key. It hands the YubiKey a small piece of data to sign (a "nonce"), the YubiKey signs internally and returns the signature. If the device is unplugged, signing is impossible regardless of what's on the laptop.

This article uses **FIDO2** (the modern protocol; SSH key types `sk-ssh-ed25519@openssh.com` and `sk-ecdsa-sha2-nistp256@openssh.com`, generated with `ssh-keygen -t ed25519-sk` or `-t ecdsa-sk`). FIDO2 has been first-class in OpenSSH since version 8.2 (February 2020). PIV — the older smartcard protocol — is covered later as an alternative.

---

## The four knobs

When you generate a FIDO2 key on a YubiKey, four properties determine how it behaves:

1. **Resident vs non-resident** — where the credential is stored.
2. **Touch** — does signing require a tap on the YubiKey?
3. **PIN** — does signing require the FIDO2 PIN?
4. **ssh-agent** — is the key loaded into ssh-agent, or used directly?

These are independent yes/no choices. Combined, they describe what it takes to sign with that particular key. The next four sections take them one at a time.

### Knob 1: resident vs non-resident

This is the one most people get wrong, so it gets the most space.

<svg viewBox="0 0 900 360" xmlns="http://www.w3.org/2000/svg" style="max-width:100%;height:auto">
  <text x="220" y="30" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="18" font-weight="700">Resident</text>
  <text x="220" y="52" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">credential lives on the YubiKey</text>
  <rect x="40" y="80" width="160" height="100" rx="6" stroke="currentColor" stroke-width="2" fill="none"/>
  <text x="120" y="105" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">~/.ssh/id_root</text>
  <text x="120" y="135" text-anchor="middle" fill="currentColor" font-family="monospace" font-size="13">ID: 0xA3F2…</text>
  <text x="120" y="160" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.55">just a label</text>
  <rect x="240" y="80" width="160" height="100" rx="6" stroke="#d4a017" stroke-width="2" fill="none"/>
  <text x="320" y="105" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">YubiKey</text>
  <circle cx="305" cy="138" r="9" stroke="#d4a017" stroke-width="2" fill="none"/>
  <rect x="314" y="135" width="26" height="6" fill="#d4a017"/>
  <rect x="334" y="135" width="3" height="10" fill="#d4a017"/>
  <text x="320" y="170" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.65">credential here</text>
  <text x="220" y="225" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">Lose the file?</text>
  <text x="220" y="246" text-anchor="middle" fill="currentColor" font-family="monospace" font-size="13">ssh-keygen -K</text>
  <text x="220" y="264" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.65">recreates it from the device</text>
  <text x="220" y="300" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13" font-weight="600">Passphrase on file?</text>
  <text x="220" y="320" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.65">pointless — file holds no secret</text>
  <line x1="450" y1="20" x2="450" y2="340" stroke="currentColor" stroke-width="1" opacity="0.25" stroke-dasharray="4 4"/>
  <text x="680" y="30" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="18" font-weight="700">Non-resident</text>
  <text x="680" y="52" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">credential split between file and device</text>
  <rect x="490" y="80" width="160" height="100" rx="6" stroke="currentColor" stroke-width="2" fill="none"/>
  <text x="570" y="105" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">~/.ssh/id_wheel</text>
  <rect x="555" y="125" width="30" height="22" rx="2" stroke="currentColor" stroke-width="2" fill="none"/>
  <path d="M561 125 v-8 a9 9 0 0 1 18 0 v8" stroke="currentColor" stroke-width="2" fill="none"/>
  <text x="570" y="172" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.65">encrypted handle</text>
  <text x="680" y="138" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="32" font-weight="300" opacity="0.5">+</text>
  <rect x="710" y="80" width="160" height="100" rx="6" stroke="#d4a017" stroke-width="2" fill="none"/>
  <text x="790" y="105" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">YubiKey</text>
  <text x="790" y="138" text-anchor="middle" fill="currentColor" font-family="monospace" font-size="13">master secret</text>
  <text x="790" y="170" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.65">derives credentials</text>
  <text x="680" y="225" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">Lose the file?</text>
  <text x="680" y="246" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">credential gone</text>
  <text x="680" y="264" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.65">re-enroll a fresh handle</text>
  <text x="680" y="300" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13" font-weight="600">Passphrase on file?</text>
  <text x="680" y="320" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.65">protects real encrypted material</text>
</svg>

**Resident** (created with `-O resident`): the credential lives on the YubiKey itself. The file in `~/.ssh/` is just a pointer — a label that says "ask the device for credential 0xA3F2…". If you delete the file, you can recreate it on any machine by running `ssh-keygen -K`, which queries the YubiKey for all its resident credentials and writes them to disk.

**Non-resident** (the default): the credential is split. The YubiKey has a master secret used to derive credentials on demand. The file on disk holds an encrypted handle. To sign, the YubiKey needs the handle from the file *plus* its own master secret. Without the file, the YubiKey doesn't know which credential to derive. Without the YubiKey, the file is gibberish.

The practical consequences:

| Question | Resident | Non-resident |
|---|---|---|
| Can the file be reconstructed from the YubiKey? | Yes (`ssh-keygen -K`) | No |
| Does losing the file matter? | No | Yes (re-enroll) |
| Does a passphrase on the file add real security? | **No** — file holds an identifier, not a secret | **Yes** — file holds the encrypted credential handle |
| Is ssh-agent needed? | No, the YubiKey *is* the agent | Usually yes, to avoid re-typing the passphrase |

The headline rule:

> **Resident keys don't need a file passphrase, because the file holds nothing secret. Non-resident keys do, because the file holds the part of the credential that isn't on the YubiKey.**

A non-resident key with a passphrase is conceptually identical to a classic passphrase-protected file SSH key — except the actual signing material never leaves the YubiKey. Same mental model, with the YubiKey as a hard-bound second factor.

### Knob 2: touch

When you sign with a key, the YubiKey can require you to physically touch the gold disc. This is "user presence" — proof that a human is at the device.

- **Touch required** (default): every signing produces a touch prompt. The YubiKey's LED blinks, you tap it, the signing completes. Failure to touch within ~15 seconds aborts the signing.
- **No touch**: signings happen automatically as long as the YubiKey is plugged in. Set with `-O no-touch-required` at generation. The server's `authorized_keys` must also have `no-touch-required` for OpenSSH to accept the signature.

You turn touch off when an operation produces many signings — Ansible across hundreds of hosts, an `rsync` of 100k files, a deploy that opens 50 sessions. None of these can realistically prompt for a touch each time.

> Disable touch **only** if you plan to use short-lived ssh-agent with **password protected non-resident** keyfile!

Touch is a defense against silent malicious signing on a host you've connected to (with agent forwarding) or on a compromised laptop you happen to be at. It is *not* a defense against device theft — someone holding the device can touch it.

### Knob 3: PIN

The YubiKey has a FIDO2 PIN, set once with `ykman fido access change-pin`. It's separate from touch.

- **PIN required** (`-O verify-required` at generation): every signing prompts for the PIN.
- **PIN not required**: signing happens without a PIN (subject to touch policy).

PIN is a defense against device theft. Touch alone doesn't help you here — the thief can touch. PIN does, because the thief doesn't know it.

The same device PIN gates `ssh-keygen -K` and FIDO2 credential management generally. **Even for credentials that don't require PIN to *sign*, the device PIN is required to *extract* them.** This becomes important in the four-mode model below.

### Knob 4: ssh-agent

ssh-agent is a small process that holds keys in memory and signs on behalf of SSH clients that ask. It exists for two reasons:

- **You don't want to re-enter a file passphrase on every connection.** Load the key once, use it many times.
- **You want agent forwarding** (`ssh -A`). Connecting to host A and then from inside that session to host B, with B able to ask your laptop's agent for signatures back through the forwarded socket.

For YubiKey-backed keys, whether you need an agent depends on the connection pattern, not just on storage:

| Connection pattern | Agent needed? |
|---|---|
| Direct SSH (`ssh host`) | No — ssh client talks to the YubiKey directly |
| ProxyJump (`ssh -J jump target`) | No — local ssh signs each hop directly |
| Agent forwarding (`ssh -A`, in-session multi-hop) | Yes — remote host needs to reach your agent |
| Non-resident key with passphrase | Yes — to avoid retyping on every connection |

ProxyJump is the modern multi-hop pattern: the local ssh client opens each connection in sequence, signing each against the YubiKey directly. Nothing is exposed on intermediate hosts. Agent forwarding is the older pattern, used when you're already inside a remote shell and need to reach further (e.g., on `host1`, running `scp host2:file ./`).

For loading resident keys into the agent (when forwarding needed), no passphrase is required:

```bash
ssh-add ~/.ssh/id_sudo  # No passphrase prompt; the file holds a reference, not encrypted material.
```

> **Never** do this with no-touch no-pin keys! They must be password protected and added to agent like `ssh-add -t 10m ~/.ssh/id_wheel`

**Touch-required keys make agent forwarding safe again.** With file-based keys, an unlocked agent signs anything it's asked to, silently, for the agent's lifetime — agent forwarding became dangerous because a compromised forwarded host could sign as you on every other host you have access to. With FIDO2 touch-required keys, every signing request from a forwarded host produces a touch prompt on your laptop. If you didn't initiate the action, you don't touch, and the signing fails. The classic "never use `-A`" advice no longer applies once credentials are hardware-backed and touch-gated.

This refines the rule:

> **Resident is the default. Non-resident is reserved for keys that must live in ssh-agent for the wheel-style mass-automation use case** — explained next.

---

## The four-mode model

A single key configuration cannot serve both rare root login and Ansible across a fleet. Different operations have different blast radius and different frequency, and they want different policies. The pragmatic answer is four keys, each a deliberate combination of the four knobs.

<svg viewBox="0 0 900 320" xmlns="http://www.w3.org/2000/svg" style="max-width:100%;height:auto">
  <text x="450" y="28" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="16" font-weight="700">Four keys, four policies, one device</text>
  <g>
    <rect x="30" y="60" width="200" height="220" rx="8" stroke="currentColor" stroke-width="2" fill="none"/>
    <text x="130" y="92" text-anchor="middle" fill="currentColor" font-family="monospace" font-size="20" font-weight="700">root</text>
    <line x1="50" y1="105" x2="210" y2="105" stroke="currentColor" stroke-width="1" opacity="0.25"/>
    <text x="130" y="135" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">resident</text>
    <text x="130" y="160" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">PIN + touch</text>
    <text x="130" y="185" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13" opacity="0.7">no ssh-agent</text>
    <line x1="50" y1="205" x2="210" y2="205" stroke="currentColor" stroke-width="1" opacity="0.25"/>
    <text x="130" y="230" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">direct root SSH login</text>
    <text x="130" y="248" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">break-glass, ceremonial</text>
  </g>
  <g>
    <rect x="240" y="60" width="200" height="220" rx="8" stroke="currentColor" stroke-width="2" fill="none"/>
    <text x="340" y="92" text-anchor="middle" fill="currentColor" font-family="monospace" font-size="20" font-weight="700">sudo</text>
    <line x1="260" y1="105" x2="420" y2="105" stroke="currentColor" stroke-width="1" opacity="0.25"/>
    <text x="340" y="135" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">resident</text>
    <text x="340" y="160" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">touch only</text>
    <text x="340" y="185" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13" opacity="0.7">no ssh-agent</text>
    <line x1="260" y1="205" x2="420" y2="205" stroke="currentColor" stroke-width="1" opacity="0.25"/>
    <text x="340" y="230" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">daily admin</text>
    <text x="340" y="248" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">PAM TOTP at host</text>
  </g>
  <g>
    <rect x="450" y="60" width="200" height="220" rx="8" stroke="#d4a017" stroke-width="2" fill="none"/>
    <text x="550" y="92" text-anchor="middle" fill="#d4a017" font-family="monospace" font-size="20" font-weight="700">wheel</text>
    <line x1="470" y1="105" x2="630" y2="105" stroke="currentColor" stroke-width="1" opacity="0.25"/>
    <text x="550" y="135" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13" font-weight="700">non-resident</text>
    <text x="550" y="160" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">no touch, no PIN</text>
    <text x="550" y="185" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13" font-weight="700">passphrase + ssh-agent</text>
    <line x1="470" y1="205" x2="630" y2="205" stroke="currentColor" stroke-width="1" opacity="0.25"/>
    <text x="550" y="230" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">NOPASSWD automation</text>
    <text x="550" y="248" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">Ansible, fleet rollouts</text>
  </g>
  <g>
    <rect x="660" y="60" width="200" height="220" rx="8" stroke="currentColor" stroke-width="2" fill="none"/>
    <text x="760" y="92" text-anchor="middle" fill="currentColor" font-family="monospace" font-size="20" font-weight="700">robo</text>
    <line x1="680" y1="105" x2="840" y2="105" stroke="currentColor" stroke-width="1" opacity="0.25"/>
    <text x="760" y="135" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">resident</text>
    <text x="760" y="160" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13">no touch, no PIN</text>
    <text x="760" y="185" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="13" opacity="0.7">no ssh-agent</text>
    <line x1="680" y1="205" x2="840" y2="205" stroke="currentColor" stroke-width="1" opacity="0.25"/>
    <text x="760" y="230" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">backups, sftp</text>
    <text x="760" y="248" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="12" opacity="0.65">stage deploys</text>
  </g>
  <text x="450" y="305" text-anchor="middle" fill="currentColor" font-family="sans-serif" font-size="11" opacity="0.6">three resident, one non-resident — non-resident only when the key needs ssh-agent</text>
</svg>

The same model as a table:

| Key     | Touch | PIN  | Storage      | File pass | ssh-agent | Use                           |
|---------|-------|------|--------------|-----------|-----------|-------------------------------|
| `root`  | yes   | yes  | resident     | no        | no        | Direct root SSH login         |
| `sudo`  | yes   | no   | resident     | no        | no        | Daily admin (TOTP at host)    |
| `wheel` | no    | no   | non-resident | **yes**   | **yes**   | NOPASSWD mass automation      |
| `robo`  | no    | no   | resident     | no        | no        | Backups, sftp, stage deploys  |

### Why three are resident and one isn't

`wheel` is the deliberate exception, for three reasons that compound:

**1. Mass automation must use ssh-agent.** Ansible across 300 hosts produces thousands of signing operations per run. A touch on each is unworkable. So `wheel` is generated `no-touch-required` AND no `verify-required`. Once it's loaded into ssh-agent (so it can be reused across the run), the agent holds the key in memory.

**2. The file on disk needs a passphrase.** It's to prevent accidental loading, and to force the operator to deliberately type something before the agent gets the key.

**3. The passphrase needs a forcing function.** ssh-keygen -K on a new machine writes resident credentials into ~/.ssh — `id_root`, `id_sudo`, `id_robo` — none needing passphrases, because they're just references to material on the device. The flow trains you that "resident-export-without-passphrase is safe."

If wheel were resident, the same command would write `id_wheel`, and you'd have to remember the one exception: passphrase this file, the others are fine. Humans don't reliably catch that exception.
Non-resident wheel is structurally outside that flow: ssh-keygen -K can't produce it, and the file you copy from your existing setup already has a passphrase. A physical equivalent: keep wheel on a separate YubiKey with a "passphrase required" sticker.

### Generation commands

```bash
# root: resident, touch + PIN
ssh-keygen -t ed25519-sk -O resident -O verify-required \
  -N "" -f ~/.ssh/id_root -C "laptop-root"

# sudo: resident, touch only
ssh-keygen -t ed25519-sk -O resident \
  -N "" -f ~/.ssh/id_sudo -C "laptop-sudo"

# wheel: non-resident, no touch, no PIN, passphrase, used via ssh-agent -t 10m
ssh-keygen -t ed25519-sk -O no-touch-required \
  -f ~/.ssh/id_wheel -C "laptop-wheel"
# Set a real passphrase when prompted.

# robo: resident, no touch, no PIN
ssh-keygen -t ed25519-sk -O resident -O no-touch-required \
  -N "" -f ~/.ssh/id_robo -C "laptop-robo"
```

`-N ""` skips the file passphrase prompt. Used for the three resident keys. `wheel` is the only one without `-N ""` — you'll be prompted, and you set a real passphrase.

### Server-side `authorized_keys`

Keys generated with `-O no-touch-required` need a matching `no-touch-required` option in `authorized_keys`, otherwise OpenSSH rejects the signature.

#### root
`/root/.ssh/authorized_keys`:

```
sk-ssh-ed25519@openssh.com AAAA... laptop-root
```

#### wheel
`~wheel/.ssh/authorized_keys`:

```
no-touch-required sk-ssh-ed25519@openssh.com AAAA... laptop-wheel
```

#### sudo
`~admin/.ssh/authorized_keys` (the daily-admin user with `sudo` privileges):

```
sk-ssh-ed25519@openssh.com AAAA... laptop-sudo
```

Pair the `sudo` key with `pam_google_authenticator.so` at the host's `sudo` PAM stack:

```
# /etc/pam.d/sudo
auth required pam_google_authenticator.so
```

Per-user TOTP secrets in `/etc/google_authenticator` (readable only by root) *can* protect from stolen YubiKey (touch is not enough for sudo). Also protects you from accidental `sudo rm -rf /` .

#### robo
`~robo/.ssh/authorized_keys` — the most-restricted, non-prod-fleet entry, constrained at source IP and forced command:

```
no-touch-required,from="10.0.0.0/8",command="/usr/local/bin/backup-shell" sk-ssh-ed25519@openssh.com AAAA... laptop-robo
```

---

## PIV — the older alternative protocol

YubiKey supports a second SSH path: PIV (Personal Identity Verification), a US-government smartcard standard that predates FIDO2 by about a decade.

PIV-on-YubiKey gives you:

- Multiple "slots" (9a, 9c, 9d, 9e, plus retired 82–95) — each holds a separate certificate and key pair.
- Three touch policies per slot: `never`, `cached` (15-second window), `always`.
- PIN policies: `default`, `once`, `always`, `never`.
- Standard X.509 certificates, which integrate nicely if your environment already uses smartcards for things like email signing, S/MIME, or government identity.

A typical setup:

```bash
# Generate ECCP256 key in slot 9a, with cached touch and PIN-once
ykman piv keys generate \
  --algorithm ECCP256 \
  --touch-policy CACHED \
  --pin-policy ONCE \
  9a /tmp/pubkey.pem

# Self-signed certificate (or sign with a corporate CA)
ykman piv certificates generate \
  --subject "CN=admin" \
  9a /tmp/pubkey.pem

# Use it directly via PKCS#11
ssh -I /usr/lib/x86_64-linux-gnu/libykcs11.so user@host

# Or load into ssh-agent
ssh-add -s /usr/lib/x86_64-linux-gnu/libykcs11.so
```

On paper, the `cached` touch policy is exactly what you want. One touch unlocks signing for 15 seconds, then it locks again — ideal for `rsync` or `scp` of many files where one logical operation triggers many SSH transactions.

In practice, the cache behavior depends on how your SSH client handles the PKCS#11 session. Different clients open and close PKCS#11 sessions differently:

- Some open the session once per `ssh` invocation and keep it open, so the cache works as advertised.
- Some open and close per cryptographic operation, which resets the cache and produces a touch prompt every signing.
- Behavior varies between OpenSSH versions, between using ssh-agent vs. direct PKCS#11, between Linux distributions and OS package builds.

For a single user on one machine, PIV with `cached` can be made to work once you've found the right combination. **For a fleet with mixed client versions across Linux, macOS, and Windows, the behavior isn't predictable.** You'll get bug reports for years and your runbooks will accumulate `if your client is X, do Y` branches.

FIDO2 sidesteps this entirely. Per-credential policy is set at generation time, OpenSSH speaks the protocol natively without PKCS#11 in the middle, and behavior is consistent across clients and platforms.

> **Use PIV if** you already have smartcard tooling, X.509 workflows, or a strong organizational reason to use the existing standard.
>
> **Use FIDO2 if** you're starting fresh and want predictable behavior across a heterogeneous fleet.

---

## Software-only alternatives

Hardware tokens cost money and procurement takes time. For distributed contractors, BYOD policies, or organizations without an IT budget for keys, you're sometimes deploying software-only solutions. The options below all keep your private key better-protected than a plain file in `~/.ssh/`, but with different trade-offs.

The dimension that matters: **can the private key be extracted from where it lives?**

| Solution            | Key storage                    | Extractable?                  | Notes                                                   |
| ------------------- | ------------------------------ | ----------------------------- | ------------------------------------------------------- |
| Secretive (macOS)   | Apple Secure Enclave           | **No**                        | Touch ID per signing. Open source.                      |
| Windows Hello SSH   | Windows TPM                    | **No**                        | TPM-bound; biometric/PIN per signing. Caveats below.    |
| KeePassXC SSH agent | Encrypted KDBX database        | **Yes** (when DB unlocked)    | Keys are read from disk; the DB is just an extra layer. |
| 1Password SSH agent | 1Password vault (cloud-synced) | **Yes** (extractable when vault is unlocked locally) | Convenient. You're trusting their infrastructure.       |
| LastPass SSH agent  | LastPass vault (cloud-synced)  | **Yes** (2022 breach; weak master passwords brute-forced offline)  | LastPass had a major vault-data breach in 2022.         |

The categories sort cleanly:

**Hardware-backed (Secretive, Windows Hello).** The private key is generated inside a secure element and never leaves it. Same security model as a YubiKey, but tied to one device. Strong for "I always work from this laptop"; weaker for "I work from three machines."

**Note on Windows Hello SSH.** "Windows Hello SSH" gets used to describe three different things, only one of which is genuinely the macOS-Secretive equivalent:

- **TPM-backed via Virtual Smart Card** — the actual TPM-bound SSH path. Requires `tpmvscmgr.exe` to create a virtual smart card, a self-signed cert via the Microsoft Smart Card Key Storage Provider, and PuTTY/Pageant rather than the default OpenSSH client. **`tpmvscmgr.exe` is Pro/Enterprise/Education only** — not available on Windows 11 Home.
- **Windows Hello for Business** — the corporate path, requires Entra ID or AD join. Out of scope for a personal laptop.
- **`ssh-keygen -t ed25519-sk` with Windows Hello as the UV layer** — the most-documented "Windows Hello SSH" path, but Windows Hello is just the UI layer asking for your PIN. The actual FIDO2 authenticator is still a USB device (typically a YubiKey). On Windows 11 Home, this is effectively the only available option, which means you need external hardware anyway.

The takeaway: on macOS, software-only hardware-backed SSH is one click in Secretive. On Windows it's an enterprise feature with awkward retrofitting, and Home users are pushed toward an external YubiKey regardless. This is one of the practical reasons a YubiKey wins on cross-platform — the same device works the same way on every OS, no per-OS puzzle to solve.

**Software-encrypted (KeePassXC).** The key is a normal SSH private key, encrypted in a database. Strictly better than a naked file because there's a master password gating access, but the key is still extractable any time the DB is open. Reasonable when you already use KeePassXC for password management.

**Cloud-synced (1Password, LastPass).** The key is stored in the provider's vault. Whoever can read the vault can read the key. You're trusting the provider's infrastructure and operational security. 1Password's design (Secret Key + master password) makes server-side decryption genuinely difficult; LastPass's 2022 breach demonstrated that vault contents can leak in practice. The convenience is real; the trust assumption is non-trivial.


Pick the strongest option you can ship to your team, and back it with a multi-mode model along the same lines as the YubiKey one — different keys for different operation classes, with the most automated keys getting the strongest restrictions at the server side.

---

## SSH CAs — Teleport, step-ca, HashiCorp Boundary

Everything above is about **credential custody**: where the private key lives and what's required to use it.

Teleport, step-ca (Smallstep's open-source CA), and HashiCorp Boundary solve a related but distinct problem: **credential lifecycle and access control**. Instead of long-lived keys, they issue short-lived SSH certificates that expire automatically. They integrate with identity providers (Okta, Google Workspace, Entra ID), log session activity, and can grant just-in-time access that revokes itself.

Whether you need this depends on scale.

| Team size | Typical reality | Recommendation |
|---|---|---|
| Solo or up to ~15 people | You know who has access. `authorized_keys` is auditable by reading. Offboarding is manual but tractable. | YubiKey + four-mode model is enough. A CA adds operational overhead without proportional security gain. |
| 15–100 people, growing | New hires need access; departures need offboarding; "who can SSH to production?" stops being answerable from `authorized_keys` alone. Onboarding takes a day per person. | Adopt a CA system. Pain is real and pays back the investment. |
| Hundreds of devs, regulated industry | Manual key management is impossible. You can't audit it, you can't rotate it, you can't prove who logged into what after the fact. | CA system is mandatory. Plan around it from day one. |

The operational pain shows up in roughly this order as you grow:

1. Adding a key to N hosts requires Ansible discipline. Doable.
2. Removing a key from N hosts requires the same discipline. Often skipped on departures.
3. Rotating keys regularly across the whole fleet is a project.
4. Answering "is this person's access still active?" requires querying every host. Expensive.
5. Proving to an auditor what happened in a session three months ago requires session logging that `authorized_keys` doesn't provide.

Each of these gets harder in a known order, and each has a CA-shaped solution.

**The common confusion: SSH CAs don't replace hardware keys. They complement them.**

When you use a CA, the long-term identity authenticates to the CA's enrollment endpoint and gets a short-lived SSH certificate in return. That long-term identity needs to be protected — if it's a file-based key, an attacker who steals it can request fresh certificates indefinitely. The CA system has moved the problem rather than solved it.

The right shape:

- **Long-term identity:** YubiKey + the four-mode model (or just `sudo`/`root` keys, depending on what the CA expects).
- **Short-term access:** SSH certificates issued by the CA, valid for hours, scoped to specific hosts.
- **Audit:** CA logs the issuance; session recording captures what happened during use.

The hardware-backed identity is the foundation. The CA is the access plane on top of it.

---

## TL;DR

The four knobs:

- **Resident vs non-resident** — where the credential lives. Resident is the default; the file is a label, no passphrase needed. Non-resident is for keys that must be in ssh-agent; the file holds encrypted material and *must* have a passphrase.
- **Touch** — physical proof of presence. Defends against silent signing on a forwarded or compromised host. Not a defense against device theft.
- **PIN** — defense against device theft. Also gates `ssh-keygen -K` extraction of resident credentials.
- **ssh-agent** — not needed for direct SSH or ProxyJump. Needed for agent forwarding (`-A`, including in-session multi-hop) and for non-resident keys with passphrases. With FIDO2 + touch-required keys, agent forwarding is safe again because every signing requires a touch on your laptop — silent signing isn't possible.

The four-mode model:

- `root` — resident, PIN + touch. Direct root login, rare.
- `sudo` — resident, touch only. Daily admin. Pair with PAM TOTP at the host.
- `wheel` — non-resident, no touch, passphrase + ssh-agent. NOPASSWD mass automation. Non-resident specifically so device + PIN cannot extract it.
- `robo` — resident, no touch, no PIN. Convenience tier, restricted at the server with `from=` and `command=`.

Other paths and where they fit:

- **PIV** is theoretically cleaner (slots, certificates, cached touch policy) but its caching depends on PKCS#11 session handling that drifts between SSH client versions. Avoid for heterogeneous fleets.
- **Software alternatives** sort by extractability. Secretive and Windows Hello are hardware-backed (non-extractable). KeePassXC, 1Password, and LastPass are extractable to varying degrees of "the provider can see your key."
- **SSH CAs** (Teleport, step-ca, HashiCorp Boundary) solve access management at scale. They don't replace hardware keys — they sit on top of them. Adopt when manual `authorized_keys` management starts hurting, typically around 15–100 engineers.

The shortest possible version: hardware key first, multi-mode policy second, CA system if and when scale demands it.
