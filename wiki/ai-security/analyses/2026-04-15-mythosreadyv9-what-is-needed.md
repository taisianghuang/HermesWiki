---
title: "Mythos-ready Security Program 所需條件分析"
type: analysis
topic: ai-security
created: 2026-04-15
updated: 2026-04-15
created_by: hermes
tags: [ai-security, mythosready, vulnops, security-program, owasp, mitre-atlas, nist-csf]
sources: [../sources/2026-04-15-mythosreadyv9.md, ../concepts/2026-04-mythos-ai-vulnerability-storm.md]
related: [Mythosreadyv9 CSA 報告來源, Mythos AI 漏洞風暴]
summary: 根據 CSA Mythosreadyv9 報告，Mythos-ready 安全計畫需要管理 13 項關鍵風險（4 項 CRITICAL）、執行 11 項優先行動（三大時間視野），並建立 VulnOps 作為長期核心能力。
---

# Mythos-ready Security Program 所需條件分析

## 核心前提：結構性非對稱

Mythos 代表的改變是**結構性的**：AI 降低攻擊方的成本與技能門檻，速度壓縮到小時級，而修補方受限於內在限制，無法瞬間修補所有系統。**這個不對稱不會逆轉**。

---

## 13 項關鍵風險（需被管理）

| 嚴重性 | # | 風險 | 類型 |
|--------|---|------|------|
| **CRITICAL** | 1 | AI-自主漏洞生成（機器速度） | Threat |
| **CRITICAL** | 2 | 防禦方缺乏 AI 自動化能力 | Capability Gap |
| **CRITICAL** | 3 | 無管理的 AI Agent 攻擊面 | Vulnerability |
| **CRITICAL** | 4 | 人力檢測/回應速度不足 | Capability Gap |
| **HIGH** | 5 | 網路安全風險模型過時 | Governance |
| **HIGH** | 6 | 資產與漏洞暴露庫不完整 | Vulnerability |
| **HIGH** | 7 | 軟體交付管道無 AI 安全審查 | Vulnerability |
| **HIGH** | 8 | 網路架構不足以限制橫向移動 | Vulnerability |
| **HIGH** | 9 | 持續漏洞管理成熟度缺口 | Vulnerability |
| **HIGH** | 10 | CVE/KEV 智能被 AI 發現速率超越 | Governance |
| **HIGH** | 11 | AI 發現漏洞的監管與責任風險 | Governance |
| **MEDIUM** | 12 | 創新治理與監督缺口 | Governance |
| **MEDIUM** | 13 | AI 炒作導致系統性無作為 | Governance |

---

## 11 項優先行動（90天執行框架）

### CRITICAL — 本週開始

**行動 1：用 Agent 掃描自身程式碼與 CI/CD 管線**
- 立即對任何程式碼請求 agent 做安全審查
- 建立 CI/CD pipeline 中的 LLM 安全審查機制
- 所有程式碼（人工或 AI 生成）都需在 merge 前通過 LLM 驅動的安全審查
- 工具：Claude Code Security、OpenAnt（Knostic）、raptor、Trail of Bits agentic skills

**行動 2：強制採用 AI Agent（安全團隊全員）**
- 將 AI agent（主要是 coding agents）定為所有安全職能的標配
- 漏洞分級、修補測試、紅隊演練、審計數據收集、安全維運加速
- 可選的 adoption 方案已被證實無法克服文化障礙

**行動 4：建立創新加速治理機制**
- 跨職能機制（安全、法務、工程）評估新威脅並加速防禦技術 onboard
- 沒有這個機制，所有其他行動都會碰到審批摩擦

**行動 5：準備持續修補能力**
- Glasswing 計畫啟動後，40 家廠商可能同時釋出大量修補程式
- 需具備修補分類與部署容量以應對這波洪流

---

### CRITICAL — 45天內

**行動 6：更新風險模型與報告**
- 重新檢視安全風險指標、報告、業務風險計算
- AI 加速時間線導致「修補視窗」、「漏洞稀缺」、「事件頻率」等假設不再成立
- 與利害關係人溝通這項挑戰

---

### HIGH — 90天內

**行動 3：保護自己的 Agent**
- Agents 是必要工具但本身不安全 — 特權存取且預設不安全
- Agent 安全重點：prompts、tool definitions、retrieval pipelines、escalation logic
- 在部署到生產環境前，定義範圍邊界、blast radius 限制、升級邏輯、人類覆寫機制

**行動 7：庫存管理與攻擊面縮減**
- 從關鍵網路暴露系統開始，建立完整覆蓋庫存（目標：45天）
- 生成真實 SBOM（軟體物料表）
- 積極關閉不需要或無法維護的功能
- 你無法修補、分割或防禦你不知道存在的東西

---

### HIGH — 6個月內

**行動 8：強化環境**
- 實施出口過濾（曾阻擋所有公開 Log4j 漏洞利用）
- 深度分割與 Zero Trust
- 鎖定依賴鏈
- 所有特權帳號強制 phishing-resistant MFA
- 每個邊界都增加攻擊者成本

**行動 9：建立欺騙能力**
- Deception 與攻擊工具和漏洞無關，識別的是 TTP
- 部署 canaries、honey tokens、行為監控、預授權 containment 行動

**行動 11：建立 VulnOps（漏洞運營）功能**
- 這是 CSA 報告的核心長期目標
- 如 DevOps 一樣運作的永久功能：
  - 持續對自身整個軟體資產進行零日發現
  - 建立自動化修補管道
  - 從一開始就圍繞分級紀律設計

---

### HIGH — 12個月內

**行動 10：建立自動化回應能力**
- 改善偵測工程與事件回應能力，使其系統化並盡可能自動化
- 資產與用戶行為分析、預授權 containment 行動、機器速度執行的回應 playbook

---

## 三大時間視野總覽

```
本週開始（CRITICAL）
├── 用 Agent 掃描自身程式碼
├── 強制採用 AI Agent
├── 建立創新加速治理機制
└── 準備持續修補能力

45天內（CRITICAL）
└── 更新風險模型與報告

90天內（HIGH）
├── 保護自己的 Agent
└── 庫存管理與攻擊面縮減

6個月內
├── 強化環境（分段/出口過濾/MFA）
├── 建立欺騙能力
└── 建立 VulnOps（CRITICAL）

12個月內
└── 建立自動化回應能力
```

---

## 關鍵框架對照

Mythos-ready 計畫需映射到這些標準：

| 框架 | 用途 |
|------|------|
| **OWASP LLM Top 10 2025** | LLM 安全風險 |
| **OWASP Agentic Top 10 2026** | 自主 Agent 安全風險 |
| **MITRE ATLAS** | AI 對抗技術 |
| **NIST CSF 2.0** | 網絡安全框架（ID/PR/DE/RS/RC） |

---

## 核心心態

> "We cannot outwork machine-speed threats. Re-prioritize, automate, and prepare for burnout."

> "Every security role is becoming an 'AI builder' role, and the barrier is lower than most people realize."

---

## See Also

- [Mythosreadyv9 CSA 報告來源](../sources/2026-04-15-mythosreadyv9.md)
- [Mythos AI 漏洞風暴概念頁](../concepts/2026-04-mythos-ai-vulnerability-storm.md)
