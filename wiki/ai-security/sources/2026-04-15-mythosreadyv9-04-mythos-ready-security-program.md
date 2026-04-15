---
title: "IV. The Mythos-ready Security Program"
type: source
topic: ai-security
created: 2026-04-15
updated: 2026-04-15
created_by: hermes
tags: [ai-security, mythos-ready, security-program, risk-register, priority-actions, vulnops]
sources: [../../raw/ai-security/mythosreadyv9.pdf]
related: [Mythosreadyv9 CSA 報告來源]
summary: Mythos-ready 安全計畫的完整框架：三大時間視野、13項關鍵風險（CRITICAL/HIGH/MEDIUM）、11項優先行動（PA1-PA11）、風險類型與框架對照。
---

# IV. The Mythos-ready Security Program

## Overview

The changing landscape requires an approach that is both operational (incident response-like) and strategic (focused on program building over time). This is implemented across **three time horizons**.

A Mythos-ready security program should achieve **minimum viable resilience**: upgrading and realigning measurements to a higher maturity level on key metrics such as cost of exploitation, early detection of compromise, and blast radius containment.

## Key Assumptions Being Challenged

Many assumptions underlying our cyber defense programs are being challenged:
- Time to exploitation has been reduced to **minutes**
- We can no longer assume a patch will be ready in time for remediation
- Incident frequency is likely to **increase**
- The CVE system may not **scale**
- Shadow IT will fragment central control as coding agents proliferate to Citizen Coders
- Threat intelligence is **lagging** behind on vulnerability discovery and exploitation

## 13 Key Risks (Draft Risk Register)

| # | Risk | Severity | Type | Framework Refs |
|---|------|----------|------|----------------|
| 1 | AI-autonomous exploit generation at machine speed | CRITICAL | Threat | AML.T0040, AML.T0043, PR.PS |
| 2 | Insufficient AI automation capabilities for defenders | CRITICAL | Capability Gap | GV.OC, GV.RM, DE.CM |
| 3 | Unmanaged AI Agent attack surface | CRITICAL | Vulnerability | LLM06, ASI02, ASI03, AML.T0047 |
| 4 | Inadequate incident detection and response velocity | CRITICAL | Capability Gap | ASI08, AML.T0047, DE.CM, DE.AE, RS.MA |
| 5 | Cybersecurity risk model outdated | HIGH | Governance | GV.OC, GV.RM, RS.CO |
| 6 | Incomplete asset and exposure inventory | HIGH | Vulnerability | ASI04, AML.T0000, ID.AM, GV.SC |
| 7 | Unsecured software delivery pipeline | HIGH | Vulnerability | LLM01, LLM05, LLM08, ASI01, AML.T0018, PR.PS, ID.IM |
| 8 | Network architecture insufficient for lateral movement containment | HIGH | Vulnerability | PR.IR, PR.PS |
| 9 | Continuous vulnerability management maturity gap | HIGH | Vulnerability | (multiple) |
| 10 | CVE- and KEV-based intelligence structurally outpaced by AI discovery rates | HIGH | Governance | ASI10, ASI06, AML.T0018, ID.RA, ID.AM, DE.CM |
| 11 | Regulatory and liability exposure from AI-discovered vulnerabilities | HIGH | Governance | AML.T0000, DE.CM, ID.RA, GV.OV |
| 12 | Innovation governance and oversight deficit | MEDIUM | Governance | GV.OC, GV.RM, GV.RR, GV.OV |
| 13 | AI hype and confusion causing systematic inaction | MEDIUM | Governance | GV.OC, GV.RM |

## Priority Actions (90-Day Plan)

### CRITICAL — This Week

| # | Action | Category | Horizon |
|---|--------|----------|---------|
| PA 1 | Point Agents at Your Code and Pipelines — Use LLM-driven security review on all code (human or AI-generated) before merge. Integrate into CI/CD. Tools: Claude Code Security, OpenAnt, raptor, Trail of Bits agentic skills. | Risk Control | This week |
| PA 2 | Require AI Agent Adoption — Formalize mandatory AI agent usage across all security functions. Agents empower staff to operate at machine speed. Optional adoption fails to overcome cultural barriers. | Operational Enabler | This week |
| PA 4 | Establish Innovation/Acceleration Governance — Cross-functional mechanism (Security, Legal, Engineering) to evaluate new offensive threats and accelerate onboarding of defensive technologies. Without this, every other action hits approval friction. | Governance | This week |
| PA 5 | Prepare for Continuous Patching — With Glasswing's 40 vendor early access program, prepare triage and deployment capacity for a potential flood of patches. | Risk Control | This week |

### CRITICAL — Within 45 Days

| # | Action | Category | Horizon |
|---|--------|----------|---------|
| PA 6 | Update Risk Models and Reporting — Review and update security risk metrics, reporting, and business risk calculations to reflect AI-accelerated exploit timelines. Pre-AI assumptions about patch windows, exploit scarcity, and incident frequency may no longer hold. | Governance | 45 days |

### HIGH — Within 90 Days

| # | Action | Category | Horizon |
|---|--------|----------|---------|
| PA 3 | Defend Your Agents — Agents are necessary but insecure by default. Audit agent harness (prompts, tool definitions, retrieval pipelines, escalation logic). Define scope boundaries, blast radius limits, escalation logic, and human override mechanisms before deploying to production. | Risk Control | 45 days |
| PA 7 | Inventory and Reduce Attack Surface — Using agents, accelerate inventory of critical internet-facing systems (45 days to full coverage). Generate real SBOMs. Aggressively shut down unneeded functionality. You cannot patch, segment, or defend what you don't know exists. | Risk Control | 90 days |

### HIGH — Within 6 Months

| # | Action | Category | Horizon |
|---|--------|----------|---------|
| PA 8 | Harden Your Environment — Implement egress filtering (blocked every public Log4j exploit). Enforce deep segmentation and Zero Trust. Mandate phishing-resistant MFA for all privileged accounts. Every boundary increases attacker cost. | Risk Control | 6 months |
| PA 9 | Build a Deception Capability — Deception is attack-tool and vulnerability independent, identifying attacks based on TTPs. Deploy canaries and honey tokens, layer behavioral monitoring, pre-authorize containment actions. | Risk Control | 6 months |
| PA 11 | Stand Up VulnOps — Long-term, build a permanent Vulnerability Operations function staffed and automated like DevOps. Owns continuous zero-day discovery across your entire software estate and establishes automated remediation pipelines. Design around triage discipline from the start. | Risk Control | 6 months |

### HIGH — Within 12 Months

| # | Action | Category | Horizon |
|---|--------|----------|---------|
| PA 10 | Build Automated Response Capability — Improve detection engineering and incident response to be systemic and, to the degree possible, autonomous. Asset and user behavioral analysis, pre-authorized containment actions, and response playbooks that execute at machine speed. | Risk Control | 12 months |

## Severity Definitions

- **CRITICAL**: Immediate exposure if unaddressed
- **HIGH**: Significant exposure or increased risk within 45 days
- **MEDIUM**: Meaningful increased risk that weakens effectiveness of higher-priority controls if unaddressed

## Risk Type Definitions

- **Threat**: External actor capability controls raise cost but cannot eliminate it
- **Vulnerability**: Internal exploitable condition addressable through remediation
- **Capability Gap**: Defensive function missing or operating below the required level
- **Governance**: Organizational or structural failure that amplifies every other risk

## The First of Many Waves

Mythos is only the first wave of future AI technology disruptions. In building a Mythos-ready program, we are not only seeking a return to equilibrium but also preparing to maintain balance for the waves ahead.

## The Human Cost

Security practitioners are facing a culture challenge. AI is simultaneously accelerating the frequency of vulnerability reports, the volume of code organizations are shipping, and the expanding attack surface. Staff at capacity must also handle cognitive intensity of integrating AI into their own workflows — often without corresponding investment in automation, tooling, headcount, or well-being.

Leaders must be clear-eyed about the human cost: burnout and attrition represent a direct operational risk. The expertise needed to navigate this transition is scarce, takes years to develop, and is not replaceable on short timescales.
