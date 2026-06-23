---
title: "10G at Home: $15 M.2 Expander Turns an Old HP Mini Into a 10GbE Test Rig"
date: 2025-12-16
draft: false
categories: ["Infrastructure"]
summary: "My ISP started upselling 10gbit XGS-PON — but how to test it without buying a 10G-capable machine? A $15 noname M.2 → PCIe expander + AQC107 NIC on an old HP G2 mini did the job. Plus the pcie_aspm=off gotcha."
---

Recently, my home internet provider started upselling a 10gbit (XGS-PON) connection. No more words, just take my money =)

But I have no suitable devices for this — how to test the difference? 10G USB-C for notebooks is too expensive, buying a new PC just for a test is madness (yeah, say XGS-PON is not =)

Luckily, I have an old HP G2 mini as my home server for backups. Aaand, a $15 noname M.2 → PCIe expander + a 10G NIC did the stuff! Seriously — I didn't expect this to work, wow =)

![10G NIC on M.2 PCIe riser inside HP EliteDesk G2 mini](/img/10g-home-m2-expander/setup-1.jpg)

**P.S.** Don't forget the `pcie_aspm=off` kernel option — my extender cable is too long to work without it.

Very tricky setup: M.2 expanders provide four PCIe lanes, but most cheap 10G NICs require eight. Also notice the PCIe version.

![HP EliteDesk G2 mini with 10GbE RJ45 sticking out](/img/10g-home-m2-expander/setup-2.jpg)

The exact equipment list:

- **Expander:** ADT R43UF (PCIe 3.0 x4)
- **NIC:** AQC107 (Aquantia/Marvell — happy on x4)

*Originally posted on [LinkedIn](https://www.linkedin.com/posts/ilya-rusalowski_recently-my-home-internet-provider-started-activity-7406763382755708928-76su).*
