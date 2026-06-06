---
title: "180 Breaches a Second: How Software Broke Its Promise, and the Radical Fix Hiding in Plain Sight"
slug: "180-breaches-a-second"
date: 2026-04-03
draft: false
tags: ["security", "software-quality", "data-breaches", "passkeys", "ai-agents", "zero-trust", "capability-security", "opinion"]
categories: ["Security"]
summary: "180 accounts are breached every second — and most of it comes down to reused passwords and missing MFA. A look at the software quality collapse behind the headlines, and why the fix is the same infrastructure-level move HTTPS once made: passkeys, on-device DLP, and capability-scoped AI agents."
---

*Why the era of "all data is public" demands the same radical fix that HTTPS once brought to open networks*

-----

In July 2024, a single missing array bounds check in CrowdStrike's Falcon Sensor [crashed 8.5 million Windows machines](https://techtrenches.dev/p/the-great-software-quality-collapse) worldwide. Airlines grounded flights. Hospitals cancelled surgeries. Emergency services went dark. The total economic damage: at least $10 billion. The root cause was not a sophisticated nation-state attack, nor a novel zero-day exploit. It was the failure to verify that a configuration file had the expected number of fields — a mistake that would earn a failing grade in an undergraduate computer science course.

In March 2026, Iran-linked hackers [published over 300 personal emails and photographs](https://www.nbcnews.com/tech/security/iranian-hackers-publish-emails-allegedly-stolen-kash-patel-rcna265490) belonging to FBI Director Kash Patel, stolen from his personal Gmail account. The method was not sophisticated either: the director's credentials had been [circulating on dark-web markets for years](https://www.safestate.com/post/fbi-director-kash-patels-personal-email-hacked-by-iran-linked-group), harvested from earlier data breaches. He had not enabled phishing-resistant authentication. The head of America's premier law enforcement agency was undone by password reuse.

These are not isolated failures. They are symptoms of a structural crisis in how the world builds, secures, and governs digital systems — a crisis with hard data behind it and, perhaps, a surprisingly elegant resolution ahead.

## I. The Numbers Behind the Collapse

The intuition that "everything is breaking more often" is not a feeling. It is a measurable trend.

In 2024, [the average minute of IT downtime cost organisations $14,056](https://zenduty.com/blog/it-outages/), and for large enterprises the figure exceeded $23,750. Across the Global 2000, outages drained approximately $400 billion annually — and the per-incident cost kept rising even as the number of major incidents saw a slight decline. The cost escalation reflects a deeper truth: modern businesses are so thoroughly fused with their technology stacks that even brief interruptions cascade through supply chains, customer relationships, and balance sheets. IT and networking-related outages [increased in 2024, reaching 23% of all impactful outages](https://uptimeinstitute.com/about-ui/press-releases/uptime-announces-annual-outage-analysis-report-2025), driven by growing complexity and the shift to cloud and colocation, according to the Uptime Institute's 2025 Annual Outage Analysis.

In November 2025, [Cloudflare suffered a major global outage](https://www.testdevlab.com/blog/software-bugs-2025) that knocked thousands of websites offline — X, ChatGPT, Spotify, Uber, and even the tools that monitor outages, since they too depend on Cloudflare. The cause: a software bug triggered by a configuration change.

On the security side, the picture is worse. Compromised accounts [surged from approximately 730 million in 2023 to over 5.5 billion in 2024](https://surfshark.com/research/study/data-breach-recap-2024) — roughly 180 accounts breached every second, according to Surfshark's annual analysis.

![The Breach Explosion: Accounts Compromised Globally](/img/180-breaches-a-second/svg2-breach-explosion.svg)

The Identity Theft Resource Center [recorded 3,332 data compromises](https://www.hipaajournal.com/u-s-data-breach-record-2025/) in the United States alone in 2025, a new record and a 79% increase over five years. [Breach notification letters reached 1.35 billion in 2024](https://www.idtheftcenter.org/post/2024-annual-data-breach-report-near-record-compromises/), a 211% increase year-over-year, driven by five "mega-breaches" each affecting more than 100 million individuals.

And the cost of each breach is substantial: [IBM's 2025 Cost of a Data Breach report](https://deepstrike.io/blog/data-breach-statistics-2025) pegged the global average at $4.44 million per incident, with the United States topping the charts at $10.22 million.

What makes these numbers particularly damning is the banality of the attack vectors. Four of the biggest breaches of 2024 — Ticketmaster, Advanced Auto Parts, Change Healthcare, and AT&T — [collectively exposed over 1.24 billion records](https://www.hipaajournal.com/1-7-billion-individuals-data-compromised-2024/). All four could have been prevented by enabling multi-factor authentication. The [Verizon Data Breach Investigations Report 2025](https://pepelac.news/en/posts/id20377-passkeys-vs-passwords-why-2026-could-be-the-tipping-point) found that only about 3% of compromised passwords met even baseline complexity requirements. Attackers do not need zero-day exploits. As [IBM's own X-Force team put it](https://www.ibm.com/think/insights/more-2026-cyberthreat-trends): "Attackers simply do not need zero-days — they just need valid credentials and a little bit of patience."

## II. The Root Cause: Incentives, Not Ignorance

The tempting explanation is that engineers have forgotten how to build reliable systems. The reality is more uncomfortable: the economic incentives have shifted decisively against quality.

The cost of poor software quality in the United States has [grown to at least $2.41 trillion](https://dev.to/esha_suchana_3514f571649c/the-hidden-24-trillion-crisis-why-software-quality-cant-wait-57ei), according to the Consortium for Information & Software Quality. Technical debt reached $1.52 trillion. [Seventy-five percent of business and IT executives](https://flowchainsensei.wordpress.com/2026/02/04/the-software-quality-and-productivity-crisis-executives-wont-address/) now expect their software projects to fail. Sixty-nine percent of developers report losing eight or more hours per week to inefficiencies — a full day of every working week consumed by the consequences of prior shortcuts. [Sixty-six percent of global organisations](https://www.clouddatainsights.com/the-cost-of-poor-software-quality-is-higher-than-ever/) admit they are at risk of a software outage within the next year.

Meanwhile, [executive attention has drifted elsewhere](https://flowchainsensei.wordpress.com/2026/02/04/the-software-quality-and-productivity-crisis-executives-wont-address/). Beginning around 2023–2024, AI vaulted to the top of C-suite priorities, displacing quality and security concerns that had been mounting for years. In 2025, Google, Amazon, Microsoft, and Meta collectively spent $380 billion on building AI tools.

Bruce Schneier, the Harvard security researcher and one of the most respected voices in the field, has been characteristically blunt. ["We're moving into a world of untrusted systems,"](https://www.ibm.com/think/insights/more-2026-cyberthreat-trends) he told IBM in a 2026 cybersecurity trends briefing, adding that even the commonly suggested solution of greater transparency "helps, assuming you have a customer base that is sophisticated enough to understand what they're seeing." In an [interview reflecting on a decade of data privacy work](https://www.threatshub.org/blog/nearly-10-years-after-data-and-goliath-bruce-schneier-says-privacys-still-screwed/), he was blunter still: "Nothing has changed since 2015. On the corporate side, companies are spying on us even more extensively. More of our data is in the cloud. And every one of us carries an incredibly sophisticated surveillance device wherever we go."

The problem is compounded by a demographic time bomb.

![The Broken Pipeline: Who Will Secure Tomorrow's Systems?](/img/180-breaches-a-second/svg5-broken-pipeline.svg)

Companies are [eliminating junior developer positions](https://codeconductor.ai/blog/future-of-junior-developers-ai/) in favour of AI coding tools, creating what some analysts call the "Junior Gap." Entry-level developer postings dropped 60% between 2022 and 2024. Salesforce's CEO announced the company would hire "no new engineers" in 2025. A [67% hiring cliff in 2024–2026](https://algeriatech.news/ai-mentorship-crisis-hollowing-out-engineering-pipeline-2026/) means 67% fewer potential engineering leaders in 2031–2036. The [Stack Overflow 2025 Developer Survey](https://algeriatech.news/ai-mentorship-crisis-hollowing-out-engineering-pipeline-2026/) found that 66% of developers' biggest frustration was AI solutions that are "almost right, but not quite," and 45% found debugging AI-generated code more time-consuming than writing code from scratch.

And the institutional response to breaches has been opacity, not accountability. In 2020, [nearly 100% of breach notifications explained the root cause](https://www.hipaajournal.com/u-s-data-breach-record-2025/). By 2025, only 30% did. Companies are not just failing to prevent breaches — they are actively concealing how the breaches occurred.

## III. A History of Paradigm Shifts

![Three Paradigm Shifts in Digital Security](/img/180-breaches-a-second/svg1-paradigm-shifts.svg)

To understand where digital security is heading, it helps to understand where it has been. The history of internet security can be read as a series of reluctant recognitions that something previously assumed to be safe was, in fact, fundamentally public.

**The First Paradigm: "All networks are public" (circa 2010–2015)**

For the first two decades of the consumer internet, the implicit assumption was that networks could be trusted — or at least that network security was someone else's problem. WiFi was trivially interceptable. WEP encryption could be cracked from a laptop in a coffee shop. LTE networks had their own man-in-the-middle vulnerabilities. Anyone sitting on the same cafe WiFi could capture login credentials, session cookies, and email contents in plaintext.

The response, when it finally came, was HTTPS — the encryption of web traffic using TLS. Let's Encrypt launched in 2015, making SSL certificates free and automated. Google began penalising unencrypted sites in search rankings. Within a few years, encrypted web traffic went from roughly 30% to over 95%.

The genius of HTTPS was that it solved the problem at the infrastructure level, not the human level. Users did not need to understand public-key cryptography. They did not need to check certificates manually. Browsers handled it automatically and started displaying warnings when encryption was absent. The fix was, in effect, invisible — and that is precisely why it worked.

**The Second Paradigm: "All data is public" (circa 2020–present)**

The current era is defined by a new, equally uncomfortable recognition: it is no longer just the network that is untrustworthy. The data itself — wherever it is stored, however it is protected — should be assumed to be accessible to adversaries.

This is not nihilism. It is the formal security doctrine known as "Assume Breach," the [foundational principle of Zero Trust architecture](https://www.paloaltonetworks.com/cyberpedia/what-is-a-zero-trust-architecture). As [Microsoft's own Zero Trust framework](https://www.microsoft.com/en-us/security/business/zero-trust) states: "Instead of assuming everything behind the corporate firewall is safe, the Zero Trust model assumes breach and verifies each request as though it originates from an open network."

But here is where the analogy with HTTPS breaks down — and where the current crisis becomes acute. HTTPS was elegant: one protocol, deployed everywhere, transparent to users. Zero Trust is [expensive, complex, and organisationally demanding](https://privicore.com/2026/03/09/why-assume-breach-must-extend-to-the-data-layer/). Even in organisations that follow Zero Trust principles, sensitive data often remains exposed once an attacker gains application-level access. The "HTTPS of data security" has not yet been found.

Or has it?

## IV. Passkeys: The HTTPS of Authentication

The most promising candidate for the first half of that fix is already being deployed at scale. [Passkeys](https://cybersecurity.nusummit.com/blog/why-passkeys-are-finally-taking-over-in-2025/) — based on the FIDO2/WebAuthn standard — replace passwords with asymmetric cryptography tied to a user's device and biometrics. There is no shared secret to steal, no password to phish, no credential to stuff. The private key never leaves the device. Authentication happens through Face ID, Touch ID, or a device PIN.

The adoption numbers are accelerating rapidly. [Google reports over 800 million accounts using passkeys](https://techpression.com/ditching-the-password-everything-you-need-to-know-about-passkeys-in-2026/) as of early 2026. Amazon saw 175 million users create passkeys in its first year of support. [Microsoft made passkeys the default](https://www.helpnetsecurity.com/2025/10/31/passkey-adoption-trends-2025/) for all new accounts in May 2025, driving a 120% increase in passwordless authentication. [Nearly 70% of consumers now hold at least one passkey](https://www.authsignal.com/blog/articles/passwordless-authentication-in-2025-the-year-passkeys-went-mainstream), up from 39% two years prior.

The parallels with the HTTPS rollout are almost exact. Like HTTPS, passkeys are more secure *and* more convenient — [login times drop by up to 17x, and success rates reach 98%](https://techpression.com/ditching-the-password-everything-you-need-to-know-about-passkeys-in-2026/) on platforms like TikTok. Like HTTPS, adoption is being driven by platform defaults: [Apple, Google, and Microsoft](https://www.programming-helper.com/tech/passkeys-passwordless-authentication-2026-fido-apple-google-microsoft-adoption) now support passkeys at the operating-system level, and NIST's updated Digital Identity Guidelines now cite synced passkeys as phishing-resistant authentication. Like HTTPS, [regulatory pressure is accelerating](https://www.authsignal.com/blog/articles/passwordless-authentication-in-2025-the-year-passkeys-went-mainstream) the transition: the UAE mandated elimination of SMS OTPs by March 2026, India follows in April 2026, the Philippines by June 2026, and the EU Digital Identity Wallet rollout happens by end of 2026.

And like HTTPS, passkeys work precisely because they remove the human from the equation.

![Anatomy of the FBI Director's Email Hack](/img/180-breaches-a-second/svg4-patel-anatomy.svg)

The FBI director's Gmail hack would have been impossible with passkeys enabled — there would have been no password to steal from old breaches, no credential to replay. The four mega-breaches of 2024 that lacked MFA would have been blocked entirely. Passkeys do not require users to choose strong passwords, remember them, or avoid reusing them. The device handles everything. [Apple even solved the migration problem](https://www.authsignal.com/blog/articles/passwordless-authentication-in-2025-the-year-passkeys-went-mainstream) by allowing passkeys to be created automatically in the background when users sign in with their password — zero friction, no extra steps.

This is the HTTPS moment for authentication. Within three to five years, passwords will be a legacy fallback — still present, but increasingly irrelevant, like HTTP sites in 2026. The credential-stuffing attack vector, which has powered the majority of account compromises for a decade, will be effectively eliminated.

## V. The Missing Half: On-Device AI as Data Loss Prevention

![Three Infrastructure Fixes, One Pattern](/img/180-breaches-a-second/svg3-three-layers.svg)

Passkeys solve the authentication problem. They do not solve the data leakage problem. The Pentagon's ["Signalgate" scandal](https://en.wikipedia.org/wiki/United_States_government_group_chat_leaks) — in which Defence Secretary Pete Hegseth shared information from a SECRET/NOFORN CENTCOM document into Signal group chats, including one containing his wife, brother, and personal attorney — was not a credential failure. It was a judgment failure. The [Pentagon Inspector General concluded](https://www.nbcnews.com/politics/politics-news/pentagons-signalgate-review-finds-pete-hegseth-violated-military-regul-rcna247023) that Hegseth violated military regulations and that the information, had it been intercepted, could have endangered American troops. No passkey prevents a human from copying classified text into the wrong app.

This is where the next paradigm shift may be emerging, and it follows the same architectural logic as HTTPS and passkeys: solve the problem at the infrastructure level, invisibly, without depending on human discipline.

Apple has already deployed the template. Its Communication Safety feature uses on-device machine learning to detect sensitive visual content in Messages, Photos, AirDrop, and FaceTime. It runs entirely on the device's neural engine. No data is sent to Apple. It is enabled by default for child accounts. The user receives a warning, not a block. It shipped with minimal controversy precisely because it was framed as protection, not surveillance.

The same architecture could be extended to text. Apple Intelligence already runs local large language models on every text field in iOS — for autocorrect, summarisation, and writing assistance. Adding a classification layer that recognises patterns associated with sensitive data (classification markings, coordinate formats, operational timing language, Social Security numbers, API keys, medical record identifiers) would be technically straightforward. The user experience would mirror Communication Safety: a gentle iOS-style sheet saying "This content may contain sensitive information" with options to proceed or review.

The behavioural economics are well-understood. Apple's analytics opt-in — presented during device setup with a default of acceptance — achieves near-universal consent because nobody unchecks it. The same default bias would drive adoption of on-device data classification. It would be opt-out, buried in Settings, and virtually nobody would disable it.

The enterprise DLP market already recognises this direction. A [2025 Cyberhaven study](https://www.articsledge.com/post/artificial-intelligence-data-loss-prevention-ai-dlp) tracking 1.1 million workers found that 11% of data pasted into ChatGPT was classified as sensitive corporate data. Gartner predicts that by 2027, 60% of enterprises will integrate DLP controls directly into their Zero Trust architecture. But traditional enterprise DLP requires Mobile Device Management — the cumbersome infrastructure that officials like Clinton, Patel, and Hegseth have consistently refused to use. On-device AI sidesteps this entirely: personal devices become secure enough without corporate management profiles.

The business incentive for Apple is substantial. If iPhones with Apple Intelligence can credibly satisfy government and enterprise security requirements without MDM, Apple eliminates the last argument for issuing locked-down government devices. Every Pentagon official, every banker, every healthcare worker becomes an iPhone customer by default.

The product would follow the HTTPS pattern: the platform provider (Apple, Google) supplies the infrastructure — the on-device AI classification engine. Organisations supply the policy — the specific patterns and classification rules. And regulators supply the forcing function — mandating phishing-resistant authentication and data-aware devices for handling sensitive information.

## VI. The Agent Paradox: When the User Is the Threat Model

Passkeys neutralise stolen credentials. On-device DLP catches careless humans. But a third category of risk is now emerging that neither fix addresses — and it may be the hardest of all.

An AI agent needs broad access to be useful. If Microsoft Copilot cannot read your emails, calendar, Slack messages, and OneDrive files, it is useless. But the moment it can read all of that, it can also be tricked into sending all of that somewhere else. The agent is not a tool the user wields; it *is* the user — with the user's identity, the user's permissions, and none of the user's judgment.

![The Agent Attack Chain: How Prompt Injection Turns AI Assistants Into Exfiltration Channels](/img/180-breaches-a-second/svg6-agent-attack.svg)

This is not a theoretical risk. It is already being exploited.

In 2025, researchers disclosed the EchoLeak attack (CVE-2025-32711) against Microsoft Copilot. An attacker sent an email containing hidden prompt-injection instructions to a victim's mailbox. When Copilot ingested the email through retrieval-augmented generation, the hidden instructions caused it to embed sensitive data — chat logs, OneDrive files, SharePoint content, Teams messages — into outbound links. Zero clicks required. The victim's own AI assistant became the exfiltration channel.

A ServiceNow vulnerability demonstrated something worse: agents can be tricked into *recruiting other agents*. Low-privileged users embedded malicious instructions in data fields. When higher-privileged AI agents later processed those fields, they recruited even more powerful agents to perform unauthorised actions — including assigning administrator roles. The attack chain was entirely autonomous.

In a controlled exercise, McKinsey's internal AI platform "Lilli" was compromised by an autonomous agent that gained broad system access in under two hours. And a Dark Reading poll found that 48% of cybersecurity professionals now identify agentic AI as the single most dangerous attack vector.

The readiness numbers are alarming. Over 80% of technical teams have moved past planning into active testing or production with AI agents — but only 14.4% report all agents going live with full security approval. Nearly half still rely on shared API keys for agent-to-agent authentication. And 82% of executives feel confident their existing policies provide adequate protection — which may be the most terrifying statistic of all, because it means the people who control budgets do not yet understand the problem.

**What is being tried — and why it is not enough**

The industry is responding with five overlapping approaches, none yet mature.

First, *treating agents as identities, not tools*. Microsoft launched Entra Agent ID, giving each agent its own identity with Conditional Access policies — so a risky agent can be blocked the same way a risky user is blocked. The idea is sound: an agent is not a feature of Slack; it is a *user* of Slack, with its own permissions, audit trail, and revocation path.

Second, *network egress controls*. NVIDIA's AI Red Team published sandboxing guidelines with mandatory restrictions: agents should not have unrestricted outbound network access, file writes should be limited to a designated workspace, and configuration files must be protected. The agent can read your data, but it physically cannot phone home.

Third, *output filtering at the gateway*. Agent-generated outputs are treated as potential exfiltration channels — data is classified by sensitivity, and agents cannot include more than a threshold amount of confidential content in any external-facing output without human approval.

Fourth, *OWASP's "autonomy is earned" principle*. In 2025, OWASP released an Agentic AI Security framework with input from over 100 experts. Its core principle: "Autonomy is a feature that should be earned, not a default setting." Agents start with zero permissions and are promoted — like a new employee.

Fifth, *human-in-the-loop checkpoints*. An agent should never be allowed to transfer funds, delete data, or change access-control policies without explicit human approval. The problem is obvious: this kills the entire point of autonomous agents.

The honest assessment is that none of these approaches fully works. The fundamental issue is categorical. When a human copies text from Signal to Telegram, you can intercept at the clipboard. When an agent reads an entire inbox, synthesises it, and sends a summary to an API endpoint, the exfiltration risk is compounded by context-window size. Modern LLMs process hundreds of thousands of tokens in a single pass. An agent can ingest an entire document repository, compress it, and include the synthesis in an outbound API call — all within one inference step.

The HTTPS analogy breaks down here because there is no single choke point. The agent *is* the user *and* the application *and* the network layer. It is like trying to apply data loss prevention to someone's brain.

**The filesystem evolution — and the missing final step**

![The Narrowing: Five Eras of Access Control](/img/180-breaches-a-second/svg7-filesystem-evolution.svg)

The deeper architectural pattern, however, suggests where a solution might eventually emerge. The history of computing can be read as a progressive narrowing of what any single compromised entity can access:

*Shared filesystem* — every process could read every file. *Per-application sandbox* — on iOS, each app receives a unique home directory randomly assigned at install, and is prevented from accessing data stored by other apps. On macOS, Apple uses "Data Vaults" to restrict access to an app's data from all other requesting processes. *Per-object encryption* — on iOS, each file has its own per-file key, wrapped with a class key, with all key handling in the Secure Enclave, never exposed to the application processor. Files carry individual protection classes — some accessible only when the device is unlocked, some after first authentication. *Identity-bound decryption* — the logical extension being pushed by Microsoft with Purview and Entra ID: documents carry their encryption and access policy with them, so even if the file leaks, only authorised identities can decrypt it.

Each step narrows the blast radius. iOS is at step three. The industry is struggling with step five: *capability-scoped agent access*.

The conceptual model has a name in computer science theory: capability-based security, dating to Dennis and Van Horn's 1966 paper and implemented in CMU's Hydra operating system. The idea: instead of asking "who are you?" and then granting access to everything that identity is authorised for, you grant specific capabilities — unforgeable tokens that give access to specific objects for specific durations. An agent would not receive "Ilya's full access." It would receive a capability token saying "read these three files, write to this one API, for the next thirty minutes."

iOS already implements a version of this: apps must declare specific entitlements to access system resources, and the kernel enforces sandbox restrictions through mandatory access control policies loaded at launch that cannot be modified at runtime. The missing piece for the agent era is extending this model to AI: when you give an MCP-connected agent access to your system, it currently inherits your identity. What it should inherit is a scoped capability — with monitored outbound channels and enforced rate limits.

The next wave of Data Security Posture Management tools is beginning to close this loop, moving beyond discovery to automatically applying encryption, adjusting access controls, and triggering DLP workflows when data risk rises — compressing the response time from days to seconds.

The full trajectory, then: shared filesystem → per-app sandbox → per-object encryption → identity-bound decryption → capability-scoped agent access.

**The trust trilemma — and why the OS must be God**

But there is a deeper question that this trajectory leaves unanswered: *who enforces the capabilities?* And the answer exposes a fundamental architectural contradiction that nobody in the industry wants to say out loud.

The model requires the operating system to be God.

This is not a bug. It is the only design that works. The reason is a trilemma. You can have any two of three properties, but not all three:

1. *Per-object encryption* — files protected from external attackers who gain physical access to storage.
2. *DLP and agent monitoring* — the OS can inspect what flows between applications and agents.
3. *End-to-end encryption between agents* — agents can communicate privately, and the OS cannot see the content.

If you have the first and third, DLP is blind — data flows between agents in ciphertext the OS cannot inspect. If you have the first and second, agents cannot hide from the OS — which is exactly the model that works. If you try to combine the second and third, you have a contradiction: the OS cannot simultaneously inspect content and be unable to see it.

Apple already made the choice. On iOS, all third-party applications are sandboxed, and they can only communicate through services explicitly provided by the operating system. iOS offers very few inter-process communication options compared to Android, deliberately minimising the attack surface. There is no mechanism for App A to talk to App B without passing through Apple's mediated channels. The OS sees everything inside its walls.

All file key handling occurs in the Secure Enclave; the file key is never directly exposed to the application processor. Even the CPU does not see the raw keys — only Apple's custom silicon does. The architecture trusts nothing except the hardware it controls.

This is precisely the traditional corporate security model, moved down one level of abstraction. In a corporate network, the security team terminates TLS at the proxy, reads everything in plaintext, re-encrypts, and forwards. Employees have no truly private channel that bypasses corporate visibility. The company is God inside the perimeter. In the OS-as-God model, the operating system mediates all inter-process communication, inspects all inter-agent data flow, enforces DLP rules, and agents have no private channel that bypasses the OS. The Secure Enclave is God inside the device.

![The Agent Trust Hierarchy](/img/180-breaches-a-second/svg8-trust-hierarchy.svg)

For agents, this creates a strict hierarchy. The Secure Enclave sits at the root, holding all cryptographic keys. The OS kernel enforces sandboxing and mediates inter-process communication. The DLP layer — Apple Intelligence or its equivalent — inspects data flows. The agent runtime operates within its sandbox, with capability-scoped permissions. And agent-to-agent communication passes through OS-mediated channels, inspectable and filterable at every step. An agent cannot establish a channel that bypasses this stack.

The per-object encryption protects against external attackers — someone who steals the device or reads the flash storage directly. But it does not protect against the OS itself. That is by design.

The uncomfortable implication is that you are not trusting "encryption" in the abstract. You are trusting Apple. Specifically, you are trusting that Apple will not abuse its position as the root of trust; that it will not be compelled by governments to insert backdoors; that the Secure Enclave will not be compromised; and that the DLP rules enforced on your device will be *your* rules, not Apple's.

This is exactly why Apple fights so hard on privacy branding. Their entire business model for this architecture depends on users trusting them as the benevolent God of the device. The moment that trust fractures — as it nearly did in 2021, when Apple proposed on-device scanning of photos for child sexual abuse material before iCloud upload, then reversed course after massive backlash — the whole architecture loses its legitimacy. Not its technical validity, but its social licence.

Is there an alternative? In theory, yes. Homomorphic encryption and secure multi-party computation could allow agents to process data without the OS ever seeing the plaintext. But this remains largely academic. The performance overhead is enormous, and no production-grade system operates at OS scale. For the foreseeable future, the choice is between a centralised root of trust and no DLP at all.

The honest comparison, then, is not between "privacy" and "surveillance." It is between two centralised trust models. Corporate IT departments are staffed by humans who get phished, reuse passwords, leave the company, and operate under variable institutional discipline. Apple's Secure Enclave is hardware. It does not get phished. It does not reuse credentials. And it is identical across two billion devices. The attack surface is radically smaller, even if the trust model is structurally identical.

It is not zero trust all the way down. It is "trust Apple's silicon, trust nothing else." And for most threat models — including the Signalgate scenario, the Patel credential hack, and the agent exfiltration problem — that is actually good enough.

The philosophical question remains: should a single company be the root of trust for all human digital communication? That is a political question, not a technical one. And it is the one nobody wants to answer.

## VII. The Human Residual

No technical solution eliminates human agency. HTTPS did not prevent users from entering credentials on phishing sites (until browsers added warnings for that too). Passkeys will not prevent users from sharing secrets verbally. On-device DLP will not prevent a determined insider from memorising classified information and writing it on a napkin.

But the history of security engineering suggests that the goal is not perfection — it is raising the cost of failure high enough that casual negligence is caught, while accepting that deliberate malice requires a different class of countermeasures. HTTPS eliminated passive network eavesdropping. Passkeys will eliminate credential reuse and phishing. On-device AI classification will eliminate casual data spillage into unclassified channels.

What remains is the irreducible human element: judgment, accountability, and institutional culture. And here, paradoxically, the very visible failures of the past few years may be producing their own corrective. When soldiers see that their commanders leak classified strike plans into family group chats, when employees see that their CEO's email gets hacked because of password reuse, the implicit trust in "the system works" evaporates. A distributed scepticism emerges — the human equivalent of "assume breach."

The military has a doctrine for this. It is called *Auftragstaktik* — mission-type tactics, originally Prussian — the principle that subordinates should understand the *intent* behind an order, not just the order itself, so they can exercise independent judgment when communications fail or leadership is compromised. The American version is "Commander's Intent": every order includes the *why*, so that when the plan falls apart (or when the order seems insane), people down the chain can think for themselves.

System fragility, it turns out, creates human vigilance. Not the ideal security posture — but perhaps a realistic one.

## VIII. What Comes Next

The technical trajectory is clear. Within three years, passkeys will be [the dominant authentication method](https://www.getmailbird.com/passkeys-email-login-what-users-need-know/) for consumer services, and passwords will begin their long retreat to legacy-fallback status. Within five years, on-device AI classification will be a standard feature of mobile operating systems, catching casual data spillage the way spam filters catch phishing emails today — imperfectly, but well enough to make the most common failures rare.

The agent security problem will take longer. Unlike passkeys or on-device DLP, there is no single protocol that makes the problem disappear — the agent's need for broad access is fundamental to its utility. The most likely path is a layered defence: capability-scoped identity, network egress controls, output classification, and behavioural monitoring — the spam-filter model rather than the HTTPS model. Within ten years, granting an AI agent your full identity will seem as reckless as browsing the web over HTTP seems today.

The institutional trajectory is less certain. The incentive structures that reward shipping speed over code quality, that replace junior engineers with AI tools while [destroying the talent pipeline](https://dev.to/rakshath/the-junior-developer-crisis-of-2026-ai-is-creating-developers-who-cant-debug-33od), that allow executives to claim "total exoneration" after inspectors general find they violated regulations — these are not technical problems, and they will not be solved by technical means.

But if the history of HTTPS is any guide, the technical fixes do not need institutional virtue to succeed. They need only to be cheaper, easier, and more convenient than the alternatives — and to be shipped as defaults by platform providers with the market power to make them ubiquitous. That is how HTTPS won. That is how passkeys are winning. And that, perhaps, is how on-device AI classification will win too.

The era of "all data is public" is not a prophecy of doom. It is a design constraint — the same way "all networks are public" was a design constraint in 2012. We did not fix networks by making WiFi secure. We fixed them by making encryption universal. We will not fix data security by making humans disciplined. We will fix it by making the infrastructure smart enough that human discipline becomes optional.

Not great, not terrible. Just engineering.

-----

-----

### Sources

**Outages & Software Quality**

- [The Great Software Quality Collapse: How We Normalized Catastrophe](https://techtrenches.dev/p/the-great-software-quality-collapse) — TechTrenches, Sep 2025
- [Biggest IT Outages of 2023–2025](https://zenduty.com/blog/it-outages/) — Zenduty
- [Uptime Institute Annual Outage Analysis 2025](https://uptimeinstitute.com/about-ui/press-releases/uptime-announces-annual-outage-analysis-report-2025) — Uptime Institute, May 2025
- [9 Biggest Software Bugs, Fails, Glitches and Outages of 2025](https://www.testdevlab.com/blog/software-bugs-2025) — TestDevLab, Dec 2025
- [The Hidden $2.4 Trillion Crisis: Why Software Quality Can't Wait](https://dev.to/esha_suchana_3514f571649c/the-hidden-24-trillion-crisis-why-software-quality-cant-wait-57ei) — DEV Community, Aug 2025
- [The Cost of Poor Software Quality Is Higher Than Ever](https://www.clouddatainsights.com/the-cost-of-poor-software-quality-is-higher-than-ever/) — CDInsights, Jun 2025
- [The Software Quality and Productivity Crisis Executives Won't Address](https://flowchainsensei.wordpress.com/2026/02/04/the-software-quality-and-productivity-crisis-executives-wont-address/) — FlowChain Sensei, Feb 2026

**Data Breaches & Security**

- [Global Data Breach Statistics: A 2024 Recap](https://surfshark.com/research/study/data-breach-recap-2024) — Surfshark, Feb 2025
- [Data Breach Statistics 2025–2026: Global Trends & Costs](https://deepstrike.io/blog/data-breach-statistics-2025) — DeepStrike
- [U.S. Data Compromises Hit Record in 2025](https://www.hipaajournal.com/u-s-data-breach-record-2025/) — HIPAA Journal, Feb 2026
- [ITRC 2024 Annual Data Breach Report](https://www.idtheftcenter.org/post/2024-annual-data-breach-report-near-record-compromises/) — Identity Theft Resource Center, Jan 2025
- [More Than 1.7 Billion Individuals Had Data Compromised in 2024](https://www.hipaajournal.com/1-7-billion-individuals-data-compromised-2024/) — HIPAA Journal, Apr 2025
- [Cybersecurity Trends 2026](https://www.ibm.com/think/insights/more-2026-cyberthreat-trends) — IBM Think (includes Bruce Schneier quotes)

**Expert Opinions**

- [Nearly 10 Years After *Data and Goliath*, Bruce Schneier Says: Privacy's Still Screwed](https://www.threatshub.org/blog/nearly-10-years-after-data-and-goliath-bruce-schneier-says-privacys-still-screwed/) — ThreatsHub, Feb 2025
- [Bruce Schneier on Security, Society and Public AI Models](https://www.threatshub.org/blog/technologist-bruce-schneier-on-security-society-and-why-we-need-public-ai-models/) — ThreatsHub, Oct 2024
- [Schneier on Security — AI Finding Zero-Days in OpenSSL](https://www.schneier.com/crypto-gram/archives/2026/0315.html) — Schneier.com, Mar 2026

**Government Breaches**

- [Iranian Hackers Publish Emails Stolen from FBI Director Patel](https://www.nbcnews.com/tech/security/iranian-hackers-publish-emails-allegedly-stolen-kash-patel-rcna265490) — NBC News, Mar 2026
- [FBI Director Patel's Email Hacked — Technical Analysis](https://www.safestate.com/post/fbi-director-kash-patels-personal-email-hacked-by-iran-linked-group) — SafeState, Mar 2026
- [United States Government Group Chat Leaks (Signalgate)](https://en.wikipedia.org/wiki/United_States_government_group_chat_leaks) — Wikipedia
- [Pentagon IG Finds Hegseth Violated Military Regulations](https://www.nbcnews.com/politics/politics-news/pentagons-signalgate-review-finds-pete-hegseth-violated-military-regul-rcna247023) — NBC News, Dec 2025
- [Pentagon Warned Against Signal Days After Leak](https://www.npr.org/2025/03/25/nx-s1-5339801/pentagon-email-signal-vulnerability) — NPR, Mar 2025

**Zero Trust & Assume Breach**

- [What Is Zero Trust Architecture?](https://www.paloaltonetworks.com/cyberpedia/what-is-a-zero-trust-architecture) — Palo Alto Networks
- [Zero Trust Security and Strategy](https://www.microsoft.com/en-us/security/business/zero-trust) — Microsoft
- [Why "Assume Breach" Must Extend to the Data Layer](https://privicore.com/2026/03/09/why-assume-breach-must-extend-to-the-data-layer/) — PriviCore, Mar 2026

**Passkeys & Passwordless Authentication**

- [Passkeys Are Finally Taking Over in 2025](https://cybersecurity.nusummit.com/blog/why-passkeys-are-finally-taking-over-in-2025/) — Aujas / NUSummit, Dec 2025
- [Ditching the Password: Passkeys in 2026](https://techpression.com/ditching-the-password-everything-you-need-to-know-about-passkeys-in-2026/) — Techpression, Dec 2025
- [Passwordless Authentication in 2025: The Year Passkeys Went Mainstream](https://www.authsignal.com/blog/articles/passwordless-authentication-in-2025-the-year-passkeys-went-mainstream) — Authsignal, Dec 2025
- [Passkeys and Passwordless Authentication 2026: FIDO2 Across Apple, Google, and Microsoft](https://www.programming-helper.com/tech/passkeys-passwordless-authentication-2026-fido-apple-google-microsoft-adoption) — Programming Helper, Jan 2026
- [Passwordless Adoption Moves from Hype to Habit](https://www.helpnetsecurity.com/2025/10/31/passkey-adoption-trends-2025/) — Help Net Security, Oct 2025
- [Passkeys vs Passwords: Why 2026 Could Be the Tipping Point](https://pepelac.news/en/posts/id20377-passkeys-vs-passwords-why-2026-could-be-the-tipping-point) — Pepelac News, Jan 2026
- [Passkeys Are Taking Over Email Logins](https://www.getmailbird.com/passkeys-email-login-what-users-need-know/) — Mailbird, Nov 2025

**AI Agents & Capability Security**

- EchoLeak Attack (CVE-2025-32711) — Microsoft Copilot prompt injection via email leading to data exfiltration
- ServiceNow Agent Vulnerability — low-privilege prompt injection causing agent-to-agent privilege escalation
- McKinsey "Lilli" AI Platform Compromise — autonomous agent gained broad system access in under two hours during controlled exercise
- [Dark Reading Poll: 48% of Cybersecurity Professionals Identify Agentic AI as Top Attack Vector](https://www.darkreading.com/) — Dark Reading, 2025
- Agent Security Readiness Survey — 80.9% in production, 14.4% with full security approval, 45.6% on shared API keys
- [Microsoft Entra Agent ID](https://www.microsoft.com/en-us/security/business/identity-access/microsoft-entra-id) — agent identity with Conditional Access policies
- NVIDIA AI Red Team Sandboxing Guide — mandatory egress, file write, and configuration protections for AI agents
- [OWASP Agentic AI Security Framework](https://genai.owasp.org/initiatives/agentic-security-initiative/) — "Autonomy is earned, not a default setting," 100+ expert contributors
- Dennis, J.B. & Van Horn, E.C. (1966) "Programming Semantics for Multiprogrammed Computations" — capability-based security model
- [Apple Platform Security: File Data Protection](https://support.apple.com/guide/security/welcome/web) — per-file encryption, Secure Enclave key management, protection classes
- [Apple Platform Security: App Sandboxing and IPC](https://support.apple.com/guide/security/welcome/web) — all third-party apps sandboxed, IPC only through OS-mediated services
- Apple CSAM Scanning Controversy (2021) — proposed on-device photo scanning, reversed after backlash; demonstrates fragility of OS-as-root-of-trust social contract

**AI, DLP & the Junior Developer Crisis**

- [What Is AI DLP? AI-Powered Data Loss Prevention Explained](https://www.articsledge.com/post/artificial-intelligence-data-loss-prevention-ai-dlp) — ArticlEdge, 2026
- [The Junior Developer Crisis of 2026: AI Is Creating Developers Who Can't Debug](https://dev.to/rakshath/the-junior-developer-crisis-of-2026-ai-is-creating-developers-who-cant-debug-33od) — DEV Community, Mar 2026
- [The AI Mentorship Crisis: Hollowing Out the Engineering Pipeline](https://algeriatech.news/ai-mentorship-crisis-hollowing-out-engineering-pipeline-2026/) — AlgeriaTech, Mar 2026
- [Junior Developers in the Age of AI](https://codeconductor.ai/blog/future-of-junior-developers-ai/) — CodeConductor, Jan 2026
