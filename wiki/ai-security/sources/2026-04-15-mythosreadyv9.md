---
title: "The 'AI Vulnerability Storm': Building a 'Mythos-ready' Security Program"
type: source
topic: ai-security
created: 2026-04-15
updated: 2026-04-15
created_by: hermes
tags: [ai-security, vulnerability-research, autonomous-exploitation, mythos, glasswing, ciso, owasp, mitre-atlas]
sources: [../../raw/ai-security/mythosreadyv9.pdf]
related: [Mythos AI 漏洞風暴]
summary: CSA 2026年4月發布的戰略簡報，記錄 Anthropic Claude Mythos 如何以自主方式發現上千個零日漏洞、生成 181 個可用漏洞，並推動 Project Glasswing 多方漏洞協調行動。提出 13 項風險與 11 項優先行動，呼籲建立 Mythos-ready 安全計畫。
---

# The "AI Vulnerability Storm": Building a "Mythos-ready" Security Program

## Basic Info

| Field | Content |
|-------|---------|
| **Source** | https://labs.cloudsecurityalliance.org/wp-content/uploads/2026/04/mythosreadyv9.pdf |
| **Raw Location** | `raw/ai-security/mythosreadyv9.pdf` |
| **Coverage** | AI 漏洞發現、自主漏洞利用、 Mythos/Glasswing 計畫、安全計畫建議 |
| **Authors** | CSA CISO Community + SANS + [un]prompted + OWASP Gen AI Security Project |
| **Version** | 0.9 — 2026年4月14日 |

## Key Findings

### 1. Mythos 的能力突破
- **181 個可用漏洞**：在 Firefox 測試中，Mythos 生成 181 個可用漏洞，Claude Opus 4.6 在同樣條件下僅成功 2 次
- **72% 漏洞成功率**
- **發現橫跨所有主流 OS 與瀏覽器**的上千個零日漏洞
- **發現年久漏洞**：包括一個 27 年歷史的 OpenBSD 漏洞
- **無需複雜 scaffolding**：單一 prompt 即可完成複雜任務

### 2. AI 漏洞發現時間線（2025-2026 加速軌跡）

| 日期 | 事件 |
|------|------|
| 2025年6月 | XBOW 登上 HackerOne 排行榜首（首個 autonomous 系統超越人類駭客） |
| 2025年8月 | Google Big Sleep 發現 20 個真實零日（FFmpeg、ImageMagick） |
| 2025年8月 | DARPA AIxCC 在 4 小時內發現 54 個漏洞（5400萬行程式碼） |
| 2025年9月 | Adkins & Evron 發出 Singularity 警告（自治漏洞利用落後約6個月） |
| 2025年11月 | 首個 AI 編排的間諜活動曝光（中國國家級組織使用 Claude Code，~30 目標） |
| 2026年2月 | Anthropic 使用 Opus 4.6 發現 500+ 高危漏洞 |
| 2026年2月 | AISLE 發現 12 個 OpenSSL 零日（CVSS 9.8，可回溯至1998年） |
| 2026年3月 | Linux kernel 漏洞報告從每週 2 個攀升至 10 個 |
| 2026年3月 | Sysdig 記錄 AI 攻擊在 8 分鐘內取得 admin 權限 |
| 2026年3月 | [un]prompted 會議 + Zero Day Clock 發布（時間到利用 < 1 天） |
| 2026年4月7日 | Claude Mythos Preview 發布 + Project Glasswing 啟動 |

### 3. Project Glasswing
- 由 Anthropic 創建的史上最大規模多方漏洞協調行動
- 對關鍵基礎設施供應商、業界夥伴和開源維護者提供 Mythos 早期訪問
- 侷限：覆蓋範圍有限，競爭對手的開源/前線模型可能在 6 個月至 1 年內追平

### 4. 13 項關鍵風險

| # | 風險 | 嚴重性 | 類型 |
|---|------|--------|------|
| 1 | AI-自主漏洞生成（機器速度） | CRITICAL | Threat |
| 2 | 防禦方缺乏 AI 自動化能力 | CRITICAL | Capability Gap |
| 3 | 無管理的 AI Agent 攻擊面 | CRITICAL | Vulnerability |
| 4 | 人力檢測/回應速度不足 | CRITICAL | Capability Gap |
| 5 | 網路安全風險模型過時 | HIGH | Governance |
| 6 | 資產與漏洞暴露庫不完整 | HIGH | Vulnerability |
| 7 | 軟體交付管道無 AI 安全審查 | HIGH | Vulnerability |
| 8 | 網路架構不足以限制橫向移動 | HIGH | Vulnerability |
| 9 | 持續漏洞管理成熟度缺口 | HIGH | Vulnerability |
| 10 | CVE/KEV 智能被 AI 發現速率超越 | HIGH | Governance |
| 11 | AI 發現漏洞的監管與責任風險 | HIGH | Governance |
| 12 | 創新治理與監督缺口 | MEDIUM | Governance |
| 13 | AI 炒作導致系統性無作為 | MEDIUM | Governance |

### 5. 11 項優先行動（90天計劃）

| # | 行動 | 類別 | 緊急性 | 時限 |
|---|------|------|--------|------|
| 1 | 用 Agent 掃描自身程式碼與 CI/CD 管線 | 風險控制 | CRITICAL | 本週開始 |
| 2 | 強制採用 AI Agent（安全團隊全員） | 作業赋能 | CRITICAL | 本週開始 |
| 3 | 保護自己的 Agent（Agent 安全） | 風險控制 | CRITICAL | 本月 |
| 4 | 建立創新加速治理機制 | 治理 | CRITICAL | 本週開始 |
| 5 | 準備持續修補能力 | 風險控制 | CRITICAL | 本週開始 |
| 6 | 更新風險模型與報告 | 治理 | CRITICAL | 45天 |
| 7 | 庫存管理與攻擊面縮減 | 風險控制 | HIGH | 90天 |
| 8 | 強化環境（網路分段、出口過濾、MFA） | 風險控制 | HIGH | 6個月 |
| 9 | 建立欺騙能力 | 風險控制 | HIGH | 6個月 |
| 10 | 建立自動化回應能力 | 風險控制 | HIGH | 12個月 |
| 11 | 建立 VulnOps（漏洞運營）功能 | 風險控制 | CRITICAL | 6個月 |

### 6. 框架對照
報告引用四大框架：
- **OWASP LLM Top 10 2025**
- **OWASP Agentic Top 10 2026**
- **MITRE ATLAS**（AI 對抗技術）
- **NIST CSF 2.0**

### 7. 核心建議摘要
> "We cannot outwork machine-speed threats. Re-prioritize, automate, and prepare for burnout."
>
> "Every security role is becoming an 'AI builder' role."

## Key Differences from Existing Notes

此報告為 2026 年 4 月的最新資料，填補了 HermesWiki 中 AI 安全威脅演進的最新時間線空白。
