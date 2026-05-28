---
title: "I was afraid of agents yolo-mode for half a year"
date: 2026-05-10
draft: false
tags: ["agents", "sandboxing", "landlock", "go", "claude-code", "codex", "gemini"]
categories: ["Tools"]
summary: "Why I built agent-landlock — a small Go wrapper that uses Linux Landlock LSM to give coding agents YOLO mode without letting them escape the project directory."
canonicalURL: "https://www.linkedin.com/posts/ilya-rusalowski_i-was-afraid-of-agents-yolo-mode-for-half-activity-7459335865316622336-Z549"
---

I was afraid of agents yolo-mode for half a year. All my systems backed up, secrets encrypted, credentials scoped (I can't force push from my daily account, etc). But I just can't stand when agent do `npm install -g` (in a golang repo 😬, when global md says always install tools to proj dir).

I tried docker, but it is slowing workflow — I have very unusual tooling, from latex and usb devices to qemu-kvm android emulators and incus clusters. Docker is ideal for software development, but too much for research and POCs. Manage whole of this is just moving my vm to docker.

Agent sandboxing with this tooling is pain. Too restrictive, constantly "please allow Unsandboxed, this is impossible if isolated".

I just need "do whatever you want, respect unix permissions (use usb if you can), but don't cross project dir!".

**agent-landlock** — small Go wrapper around Claude Code / Codex / Gemini that uses Linux Landlock LSM (kernel 6.2+) to make host filesystem read-only for the agent process, except `$PWD` and paths you grant explicitly.

No containers, no namespaces, no paired UID, no mount tricks. Process-local, kernel cleans up when process exits. Reads still work everywhere your user can read, so LSP, git, USB, GPU, qemu-kvm, host networking all keep working.

```sh
agent-landlock claude
agent-landlock codex exec ...
agent-landlock gemini
agent-landlock run -- pytest -x
```

Forces YOLO flags by default. Persistent grants via `agent-landlock grant ~/.avd`. Fails closed if Landlock unavailable.

Golang, MIT — <https://github.com/sintoniastrategy/agent-landlock>
