---
title: "ykvault: Stop Storing API Tokens as Plaintext"
date: 2026-06-06
draft: false
tags: ["yubikey", "security", "secrets", "cli", "go", "hardware-keys"]
categories: ["Security"]
summary: "Are you still keeping API tokens in ~/.secrets? Any app you install can read them. ykvault encrypts every secret with a YubiKey challenge-response key — each get/set requires a physical touch, and the encrypted files are useless without your key."
---

Are you still storing API tokens as plaintext in `~/.secrets`? Then any application you install — and any malware that comes with it — can silently read them. On macOS and Windows, "disk access" is one careless click away.

So I built [ykvault](https://github.com/sintoniastrategy/ykvault) — a tiny CLI vault where every secret is encrypted with a key derived from your YubiKey ([original LinkedIn post](https://www.linkedin.com/posts/ilya-rusalowski_are-you-still-store-api-tokens-as-plaintext-activity-7422630349165301761-gEum)). It started as a 90-line script and grew into a small Go tool.

```bash
echo "my_api_key" | ykvault set s3-token    # store (touch required)
ykvault get s3-token                         # retrieve (touch required)
ykvault ls                                   # list secret IDs
ykvault rm s3-token                          # delete
```

### How It Works

No master password, no cloud, no background daemon. Just YubiKey HMAC-SHA1 challenge-response:

1. The secret ID (e.g. `s3-token`) is the **challenge**.
2. The YubiKey computes HMAC-SHA1 of it with a **non-extractable hardware secret** — physical touch required.
3. SHA-256 of the response → AES-256 key; SHA-256(response‖"iv")[:16] → IV.
4. AES-256-CBC encrypts the value into `~/.ykvault/<id>.ykv.slot<N>` (mode 0600).

Decryption is fully deterministic — nothing extra to store or lose. The encrypted files are useless without your YubiKey.

Setup:

```bash
ykman otp chalresp 2 --touch --generate     # one-time YubiKey config
go install github.com/sintoniastrategy/ykvault@latest
```

### Why Not Bitwarden / KeePassXC CLI?

They're fine — and more featureful. But for the narrow "give my script a token, with a hardware touch" case, ykvault is one binary, zero config, and no vault password that malware can keylog.

### The Honest Caveat

This is **hiding secrets, not securing them** — like moving from `passwords.txt` to Bitwarden. Touch-gating stops malware from *silently* scraping everything at rest, but if active malware can capture the plaintext at the moment you use it, it's already game over. Secret IDs (filenames) aren't encrypted either — only values.

For SSH specifically, go further: keys that never leave the hardware — see [Hardware-Backed SSH Keys](/hardware-backed-ssh-keys/).
