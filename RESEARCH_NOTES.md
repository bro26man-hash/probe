# 🎙️ Podcast Research Notes: OONI Probe — Digital Rights, Surveillance & the Ethics of Measuring Censorship

> **Project:** [OONI Probe](https://github.com/ooni/probe) (forked from `ooni/probe`)
> **License:** BSD-3-Clause | **Stars:** 919 | **Forks:** 149 | **Open Issues:** 370+
> **Tagline:** *"Free and open source software designed to measure internet censorship and other forms of network interference."*
> Used in **200+ countries**, with **millions of network measurements** published since 2012.

---

## 1. PROJECT OVERVIEW

OONI Probe is the flagship tool of the **Open Observatory of Network Interference (OONI)**, a project under the Tor Project umbrella. It's a free, open-source network measurement tool that_detects_-censorship, website blocking, DNS tampering, and other forms of state and corporate network interference.

**Key facts:**
- Available on Android, iOS, and Desktop (Google Play, F-Droid, App Store)
- CLI version (`probe-cli`) written in Go for advanced users
- Tests websites from the [citizenlab/test-lists](https://github.com/citizenlab/test-lists/) repository
- Has earned the **[Awesome Humane Tech](https://github.com/humanetech-community/awesome-humane-tech)** badge — a rare recognition for tools built specifically to defend human rights online
- Data is publicly available at [ooni.org/data](https://ooni.org/data/)

---

## 2. CORE SOCIETAL CONCERNS

### 2.1 The Privacy–Accuracy Paradox
**Source:** [Issue #1581 — "Privacy preserving richer geolocation"](https://github.com/ooni/probe/issues/1581) (open since 2018, 7 comments, 3 👍)

**The tension:** OONI Probe currently collects only **country-level** geolocation data to protect user privacy. But censorship is increasingly **regional** — a specific province, city, or district may block content while the rest of the country doesn't. Without granular location data, measurements can't reflect this reality.

**Community debate highlights:**
- Should the app request GPS permissions? A user in a censored region might need to prove they're in that region.
- Should there be **two app versions** — one with location, one without — for people who fear that even requesting GPS permissions could make them targets?
- Could **coarse GPS or cell tower data** provide enough accuracy without identifying individuals?
- The MaxMind vs. db-ip.com disagreement: even commercial geolocation databases disagree about which region a user is in, introducing "noise" that could misrepresent censorship patterns.

**Real-world urgency:** A user from India pointed to the **2021 Punjab mobile internet blackout**, where the government shut down mobile internet citing "security reasons." People in that region couldn't access banking, social welfare, or health services. Country-level data would completely obscure this.

**🎙️ Podcast angle:** *When you build a tool to expose censorship, you need to know WHERE the censorship is. But asking "where are you?" — even in coarse terms — creates a privacy risk for the very people you're trying to protect. How do you measure oppression without creating new vectors for identification?*

---

### 2.2 The Circumvention Cat-and-Mouse Game
**Source:** [Issue #2711 — "Add proxyless censorship resilience"](https://github.com/ooni/probe/issues/2711) (open since 2024, 7 comments)

**The reality:** OONI Probe's reporting API **has been blocked in China**. The project currently uses Tor and Psiphon as fallbacks, but a commenter proposed integrating the **Outline SDK** (from Jigsaw/Google) to provide "proxyless" censorship resistance using:
- **Encrypted DNS resolvers** (to bypass DNS-based blocking)
- **TLS Record Fragmentation** (to bypass SNI-based blocking — works in Russia, China, and Iran)
- **Happy Eyeballs strategy** (trying multiple connection methods simultaneously)

**The deeper issue:** One commenter noted that **TLS Record Fragmentation doesn't actually work in China anymore**, and that 11 domains (including The Guardian, Reddit, and The New Yorker) are blocked at the ASN level (AS56040 = China Unicom). The censorship infrastructure evolves faster than the open-source community can respond.

**🎙️ Podcast angle:** *The tools designed to circumvent censorship are themselves being censored. OONI Probe literally cannot report its findings from inside China. What does it mean when a measurement tool is blocked by the very system it's trying to measure? And when "proxyless" circumvention techniques work for a few months before being defeated, are we building democracy technology or just delaying the inevitable?*

---

### 2.3 Looking Like a User to Catch a User
**Source:** [Issue #701 — "Bridge reachability tests should also perform traffic"](https://github.com/ooni/probe/issues/701) (open since 2016, 8 comments)

**The technical dilemma:** In **Kazakhstan**, the obstm4 proxy protocol isn't blocked at the initial handshake — it's blocked only after **~50KB of traffic** has flowed. This means OONI Probe's bridge reachability test, which only checks if a connection can be established, would **miss** this type of censorship entirely. To detect it, the tool would need to actually **send real traffic** through the bridge.

But here's the problem: OONI Probe doesn't ship the Tor binary (too large, too many dependencies for mobile). The community debated using:
- **obfs4 as a library** (already done internally)
- **ptadapter** or **Shapeshifter Transports** to interact with pluggable transports
- A lightweight Go tool that mimics Tor's behavior
- Simply performing repeated TLS handshakes (but this changes the traffic fingerprint and might not trigger DPI systems trained to identify real Tor traffic)

**The ethical core:** To accurately detect censorship, the measurement tool must **mimic real user behavior**. But that makes the measurement tool look exactly like... a real user. If a government is identifying Tor users by their traffic patterns, a tool that generates realistic Tor-like traffic could itself become a **target**.

**🎙️ Podcast angle:** *To catch you, the surveillance system has to look like you. OONI Probe's developers face a profound question: if your measurement tool becomes indistinguishable from a dissident's browsing, does it become a target? Is "looking like a user" a feature or a vulnerability? And when the tool generates traffic that could be fingerprinted as censorship-evading, who bears the risk — the developer or the user?*

---

### 2.4 Encryption as Both Shield and Blindfold
**Source:** [Issue #2857 — "Add Settings toggle to warn users if they have DoH/DoT enabled"](https://github.com/ooni/probe/issues/2857) (open since 2025, 9 comments)

**The paradox:** A user noticed that when **DNS-over-HTTPS (DoH)** or **DNS-over-TLS (DoT)** is enabled, OONI Probe reports websites as **accessible** that are actually being blocked via DNS tampering. This is because encrypted DNS bypasses the very DNS manipulation that censorship systems use to block sites.

The request: add a **settings toggle** (similar to the existing VPN warning) that alerts users when secure DNS is active, because it produces **misleading results**.

**The deeper concern:** This reveals a fundamental tension in the encryption arms race. Encryption protects privacy, but it can also **blind** censorship-detection tools. When everything is encrypted, the censor's methods change (they move from DNS tampering to SNI blocking, IP blocking, or traffic shaping), but the measurement tool might not detect the new method.

**🎙️ Podcast angle:** *Encryption is supposed to be the great equalizer — it protects journalists, activists, and ordinary users alike. But what happens when encryption also hides censorship from the tools designed to expose it? If DoH makes OONI Probe "see" a free internet that doesn't exist, are we building a tool that documents reality or a tool that documents our own blindness?*

---

### 2.5 The Ethics of Community-Led Surveillance
**Source:** [Issue #832 — "Coordinate testing with friends" (a.k.a. OONI Run)](https://github.com/ooni/probe/issues/832) (open since 2019, 6 comments)

**The concept:** OONI Run is a feature that lets users **coordinate measurement campaigns** with friends, colleagues, or communities. Instead of testing from a single location, a group can simultaneously test from multiple countries, cities, or even specific institutions.

**The ethical questions:**
- What does it mean to **organize** people to run censorship-detection software? Could merely being listed as an "OONI Run coordinator" make someone a target?
- If a university in Iran or a journalist collective in Russia coordinates a test, is the act of **organizing** itself a form of political speech that could be criminalized?
- The community debated whether to integrate this prominently in the app or bury it behind a settings page — should the tool be **easy to find** (maximum impact, maximum risk) or **obscure** (safety through obscurity)?

**🎙️ Podcast angle:** *OONI Run turns censorship detection into a collective action. But collective action is precisely what authoritarian regimes are most afraid of. When you invite your friends to run a network test, are you building a community of digital rights advocates — or creating a list of suspects?*

---

## 3. ETHICAL TENSIONS SUMMARY

| Tension | Description | Podcast Potential |
|---|---|---|
| **Privacy vs. Accuracy** | Collecting location data helps measure censorship but endangers users | ⭐⭐⭐ High — universally relatable |
| **Circumvention vs. Detection** | Tools that circumvent censorship are themselves censored | ⭐⭐⭐ High — dramatic cat-and-mouse |
| **Mimicry vs. Exposure** | Looking like a user to detect blocking makes you look like a user to censors | ⭐⭐⭐⭐ Critical — existential risk |
| **Encryption vs. Transparency** | Encryption shields users but blinds measurement tools | ⭐⭐ Medium — nuanced technical |
| **Community vs. Safety** | Organizing users maximizes impact but creates target lists | ⭐⭐⭐ High — political organizing angle |
| **Open Source vs. Operational Security** | Public code is transparent but also reveals detection methods to censors | ⭐⭐ Medium — insider knowledge |

---

## 4. KEY NARRATIVE THREADS FOR THE PODCAST

### Thread 1: "The Measurement Paradox"
Every censorship-detection tool faces the same diamond problem: **to measure censorship accurately, you must collect data that could identify you; to protect yourself, you must collect less data, which makes your measurements less accurate.** OONI Probe's geolocation debate (Issue #1581) is the purest expression of this paradox. The team has been debating it for **7 years** with no resolution.

### Thread 2: "The Censor's Arms Race"
The conversation in Issue #2711 reveals that censorship circumvention is not a solved problem — it's a **living, evolving conflict**. When one technique works (TLS fragmentation), it works for a window of time before being defeated. The open-source community is essentially playing **whack-a-mole with nation-state adversaries**, and the mole always has more teeth.

### Thread 3: "The Double Life of Measurement Tools"
Issue #701 highlights that OONI Probe must generate traffic that **looks like real user traffic** to detect advanced blocking. This means the tool occupies a double life: it's simultaneously a **scientific instrument** (measuring network behavior) and a **potential camouflage** (looking like a dissident's connection). This duality creates legal and ethical gray zones.

### Thread 4: "The Blindness Paradox"
Issue #2857 reveals that the very encryption we celebrate can **obscure censorship from view**. When DNS queries are encrypted, the censor switches tactics — but the measurement tool might still be looking for the old tactics. The tool becomes **blind to the censorship it's designed to find**. This is a profound metaphor for how privacy tools can sometimes create false senses of security.

### Thread 5: "Who Watches the Watchmen?"
The OONI Run feature (Issue #832) raises the question of **who is gathering the data about censorship, and what happens to them**. OONI is a project of the Tor Project, which itself is funded by governments, NGOs, and individual donors. The funding sources create their own ethical questions — does the Tor Project's funding from the U.S. government affect which censorship stories get told?

---

## 5. EXPERT SOURCES & FURTHER READING

| Source | Why It Matters |
|---|---|
| [OONI Probe README](https://github.com/ooni/probe) | Official project description and scope |
| [Issue #1581 Discussion](https://github.com/ooni/probe/issues/1581) | The privacy-accuracy paradox, community meeting notes |
| [Issue #2711 Discussion](https://github.com/ooni/probe/issues/2711) | Circumvention techniques, China blocking, ASN-level censorship |
| [Issue #701 Discussion](https://github.com/ooni/probe/issues/701) | Traffic-based blocking, Kazakhstan, DPI fingerprinting |
| [Issue #2857 Discussion](https://github.com/ooni/probe/issues/2857) | DoH/DoT blindness, DNS tampering detection |
| [Issue #832 Discussion](https://github.com/ooni/probe/issues/832) | Community organizing, OONI Run, collective measurement |
| [OONI Explorer](https://ooni.org/explorer) | Interactive data visualization of global censorship measurements |
| [OONI Data](https://ooni.org/data/) | Publicly available measurement data (millions of tests) |
| [Citizen Lab Test Lists](https://github.com/citizenlab/test-lists) | Curated lists of websites tested for censorship |
| [Outline SDK (Jigsaw)](https://github.com/Jigsaw-Code/outline-sdk) | Proxyless circumvention techniques discussed in issues |
| [TLS Record Fragmentation Research](https://upb-syssec.github.io/blog/2023/record-fragmentation/) | Academic paper on the technique discussed in Issue #2711 |
| [India Punjab Blackout](https://therecord.media/india-punjab-mobile-internet-blackout) | Real-world example cited in issue comments |

---

## 6. SUGGESTED PODCAST STRUCTURE

1. **Cold Open:** "In Kazakhstan, a censorship system doesn't block your connection — it waits until you've sent 50KB of data, then blocks you. And the tool designed to catch this has been debating how to detect it for 8 years."

2. **Act 1 — The Mission:** What is OONI Probe, why does it exist, and what has it detected? (200+ countries, millions of measurements)

3. **Act 2 — The Paradoxes:** The five ethical tensions explored above, using real issue discussions

4. **Act 3 — The Arms Race:** How censorship techniques evolve (DNS tampering → SNI blocking → traffic-based blocking) and how circumvention techniques respond (but always a step behind)

5. **Act 4 — The Human Cost:** The Punjab blackout, the Chinese blocking of OONI's own API, the risk to community organizers

6. **Closing:** "Every measurement OONI Probe takes is a small act of defiance. But every act of measurement is also an act of exposure. And that tension — between seeing and being seen — is at the heart of digital rights."

---

## 7. QUOTABLE MOMENTS FROM THE CODE & ISSUES

> *"We have a responsibility to protect user privacy to the extent that it's possible and therefore we don't collect IPs or store location information that is more granular than country level."* — @hellais, Issue #1581

> *"In places such as Kazakhstan obfs4 is blocked not on the initial handshake, but only after some amount of traffic is done (rumor has it that the magic number is 50KB)."* — @hellais, Issue #701

> *"The emergence of regional censorship efforts is making it even more urgent for us to solve this problem."* — @bassosimone, Issue #1581

> *"OONI Probe never uses domain fronting (and hardcoded IPs?) for some APIs... 11 domains in it are blocked in AS56040"* — @Lanius-collaris, Issue #2711

> *"I am a user in a country where a specific region has increased levels of censorship and I want this fact to be reflected in my measurements."* — @bassosimone, Issue #1581

---

*Notes compiled from GitHub issues, repository metadata, and community discussions. Forked from `ooni/probe` for research purposes. All issues referenced remain open in the upstream repository.*
