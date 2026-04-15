---
title: "III. Introduction"
type: source
topic: ai-security
created: 2026-04-15
updated: 2026-04-15
created_by: hermes
tags: [ai-security, introduction, mythos, glasswing, zero-day-clock, ai-evolution-timeline]
sources: [../../raw/ai-security/mythosreadyv9.pdf]
related: [Mythosreadyv9 CSA 報告來源]
summary: 介紹 Mythos 的突破性能力（三項關鍵技術特徵）、Project Glasswing 的多方漏洞協調行動，以及 2025-2026 年 AI 漏洞發現能力的加速軌跡時間線。
---

# III. Introduction

## The New Reality

Many of our assumptions about the capabilities of AI in vulnerability research, exploitation, and autonomous attacks may be outdated. Throughout 2025 and into 2026, we've seen continuous examples of increasing capabilities, both in research and in actual in-the-wild attacks.

AI-driven vulnerability discovery and exploitation has been accelerating for over a year. See Appendix A for more details and historical evidence.

Anthropic's Claude Mythos (Preview) represents a step change in that trajectory, autonomously finding thousands of critical vulnerabilities across every major operating system and browser, generating working exploits without human guidance, and empowering autonomous attack orchestration, all at a speed and scale that outpaces any prior capability.

## The Structural Asymmetry

The asymmetry this creates is structural. AI lowers the cost and skill floor for discovering and exploiting vulnerabilities faster than organizations can patch them. The window between discovery and weaponization has collapsed to hours. Attackers gain disproportionate benefit, and current patch cycles, response processes, and risk metrics were not built for this environment.

## What Makes Mythos Different

While many of these capabilities pre-date this new model, Mythos-class capabilities do represent a step-change, and will proliferate. The organizations that respond well will be those that build the muscle now: the processes, the tooling, and a culture willing to adopt AI as a core part of how security gets done.

Mythos exhibits three capabilities that make it different:

**Complex, chained vulnerabilities**: Mythos identifies vulnerabilities composed of multiple primitives chained together, such as scenarios requiring multiple memory corruption bugs combined into a single exploit path.

**"One-shot" (single-prompt) capability**: Mythos accomplishes significantly more with a single prompt, without elaborate scaffolding or agent configuration.

## Mythos & Glasswing: Why They Matter

### Mythos

Mythos is distinguished from previous capabilities on both technological and strategic levels. Technologically:

- **Exploits without scaffolding**: Internal lab environment testing at Anthropic showed Mythos generated **181 working exploits on Firefox** where Claude Opus 4.6 succeeded only twice under the same conditions, marking a substantial jump in autonomy and reliability.

### Project Glasswing

The scale and speed of Mythos prompted Anthropic to create Project Glasswing, possibly the **largest multi-party vulnerability coordination effort in history**. Anthropic provided selected critical infrastructure providers, industry partners, and open source maintainers early access to Mythos so they could patch their own products.

The **most significant limitation** of Project Glasswing is that it can only cover so much. The world's exploitable attack surface is vastly larger than what any curated partner ecosystem can cover.

While the coordination model Glasswing established is critically important, its impact will depend heavily on how quickly it can expand coverage, and whether the patch and disclosure pipeline can keep pace with both AI progress and adversarial adoption.

## The Evolution of LLM-based Offensive Capabilities (2025/2026)

| Date | Event |
|------|-------|
| Jun 24, 2025 | XBOW tops the HackerOne leaderboard — first autonomous system to outperform all human hackers on the platform |
| Aug 5, 2025 | Google's Big Sleep finds 20 real-world zero-days in FFmpeg, ImageMagick |
| Aug 8, 2025 | DARPA AIxCC finals at DEF CON 33 — 54 vulnerabilities in 4 hours across 54 million lines of code |
| Sep 2025 | Adkins & Evron publish Singularity warning — autonomous vulnerability discovery ~6 months away |
| Nov 14, 2025 | First AI-orchestrated espionage campaign — Chinese state-sponsored group used Claude Code across ~30 global targets |
| Feb 5, 2026 | Anthropic reports 500+ high-severity vulnerabilities in open source software |
| Feb 5, 2026 | AISLE found 12 OpenSSL zero-days, including a CVSS 9.8 flaw dating to 1998 |
| Mar 2026 | Linux kernel bug reports climbed from 2 to 10/week (initially hallucinated, now all verified real) |
| Mar 2026 | Sysdig documented AI-based attack reaching admin-level access in 8 minutes |
| Mar 2026 | [un]prompted conference + Zero Day Clock launched — time-to-exploit now under one day |
| Apr 7, 2026 | Claude Mythos Preview + Project Glasswing launched — thousands of zero-days, 72% exploit success rate, 27-year-old OpenBSD bug found |
