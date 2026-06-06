---
title: "Another Ubuntu Bug With a Stopgap Fix: apt-get update Hangs for Hours"
date: 2026-02-13
draft: false
tags: ["ubuntu", "apt", "linux", "unattended-upgrades", "security", "ops"]
categories: ["Linux"]
summary: "apt-get update randomly hangs for hours on Ubuntu 24.04 LTS — known since 22.04, still not fixed. Worst part: it silently blocks unattended-upgrades, so your servers stop receiving security updates. The 'fix' is a cron job that kills stuck apt processes."
canonicalURL: "https://www.linkedin.com/posts/ilya-rusalowski_another-bug-in-ubuntu-with-a-stopgap-fix-activity-7427388465144705024-LF2M"
ShowCanonicalLink: true
---

Another bug in Ubuntu with a stopgap fix. Not a single year without them.

Latest: `apt-get update` randomly hangs for hours on Ubuntu 24.04 LTS. Known since 22.04, still not fixed.

The worst part? It silently blocks **unattended-upgrades**. Your servers stop receiving security updates and you don't even know (if you have no alerts 🥲).

The "fix"? A cron job that kills stuck apt processes 🙄.

That's why Google, Amazon, and others roll their own Linux distributions.
