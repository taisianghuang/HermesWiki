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

# IV. The Mythos-ready Security Program（Mythos-ready 安全計畫）

## 概述

不斷變化的格局及相關風險和影響，要求一種既有操作性（類似事件回應）又有策略性（專注於長期計畫建設）的方法。這透過三個時間視野來實施。

Mythos-ready 安全計畫應實現**最低可行韌性**：在關鍵指標上將測量升級並重新對齊到更高成熟度，例如漏洞利用成本、妥協早期檢測和爆炸半徑控制。

## 被挑戰的關鍵假設

支撐我們網路防禦計畫的許多假設正在被挑戰：
- 漏洞利用時間已壓縮到**分鐘級**
- 我們不能再假設修補能在補救目的所需的時間內完成
- 事件頻率可能會**增加**
- CVE 系統可能無法**擴展**
- 隨著 coding agent 普及到公民開發者，影子 IT 將分散中央控制
- 威脅情資在漏洞發現和利用方面**落後**

## 13 項關鍵風險（風險登記表草案）

| # | 風險 | 嚴重性 | 類型 | 框架參考 |
|---|------|--------|------|---------|
| 1 | AI-自主漏洞利用生成（機器速度） | CRITICAL | Threat | AML.T0040, AML.T0043, PR.PS |
| 2 | 防禦方缺乏足夠 AI 自動化能力 | CRITICAL | Capability Gap | GV.OC, GV.RM, DE.CM |
| 3 | 無管理的 AI Agent 攻擊面 | CRITICAL | Vulnerability | LLM06, ASI02, ASI03, AML.T0047 |
| 4 | 事件檢測和回應速度不足 | CRITICAL | Capability Gap | ASI08, AML.T0047, DE.CM, DE.AE, RS.MA |
| 5 | 網路安全風險模型過時 | HIGH | Governance | GV.OC, GV.RM, RS.CO |
| 6 | 資產和暴露庫存不完整 | HIGH | Vulnerability | ASI04, AML.T0000, ID.AM, GV.SC |
| 7 | 軟體交付管道不安全 | HIGH | Vulnerability | LLM01, LLM05, LLM08, ASI01, AML.T0018, PR.PS, ID.IM |
| 8 | 網路架構不足以限制橫向移動 | HIGH | Vulnerability | PR.IR, PR.PS |
| 9 | 持續漏洞管理成熟度缺口 | HIGH | Vulnerability | （多框架）|
| 10 | CVE 和 KEV 情資被 AI 發現速率結構性超越 | HIGH | Governance | ASI10, ASI06, AML.T0018, ID.RA, ID.AM, DE.CM |
| 11 | AI 發現漏洞的監管和責任暴露 | HIGH | Governance | AML.T0000, DE.CM, ID.RA, GV.OV |
| 12 | 創新治理和監督缺口 | MEDIUM | Governance | GV.OC, GV.RM, GV.RR, GV.OV |
| 13 | AI 炒作和困惑導致系統性無作為 | MEDIUM | Governance | GV.OC, GV.RM |

## 優先行動（90天計畫）

### CRITICAL——本週開始

| # | 行動 | 類別 | 時限 |
|---|------|------|------|
| PA 1 | **用 Agent 掃描自身程式碼和管線**——在 merge 前對所有程式碼（人工或 AI 生成）使用 LLM 驅動的安全審查。整合到 CI/CD。工具：Claude Code Security、OpenAnt、raptor、Trail of Bits agentic skills。 | Risk Control | 本週 |
| PA 2 | **強制採用 AI Agent**——將 AI Agent 使用定為所有安全職能的強制性標配。Agent 賦能員工以機器速度運作。可選的 adoption 方案已被證實無法克服文化障礙。 | Operational Enabler | 本週 |
| PA 4 | **建立創新加速治理機制**——跨職能機制（安全、法務、工程）評估新攻擊性威脅並加速防禦技術的 onboard。如果沒有這個機制，所有其他行動都會碰到審批摩擦。 | Governance | 本週 |
| PA 5 | **準備持續修補能力**——隨著 Glasswing 的 40 家廠商早期准入計畫，準備分類和部署容量以應對潛在的大量修補洪流。 | Risk Control | 本週 |

### CRITICAL——45天內

| # | 行動 | 類別 | 時限 |
|---|------|------|------|
| PA 6 | **更新風險模型和報告**——審查並更新安全風險指標、報告和業務風險計算，以反映 AI 加速的漏洞利用時間線。關於修補視窗、漏洞稀缺性和事件頻率的 AI 前假設可能不再成立。 | Governance | 45天 |

### HIGH——90天內

| # | 行動 | 類別 | 時限 |
|---|------|------|------|
| PA 3 | **保護你的 Agent**——Agent 是必要的但預設不安全。審計 Agent 套具（prompts、tool definitions、retrieval pipelines、escalation logic）。在部署到生產環境之前，定義範圍邊界、爆炸半徑限制、升級邏輯和人類覆寫機制。 | Risk Control | 45天 |
| PA 7 | **庫存管理和攻擊面縮減**——使用 Agent，加速關鍵網路暴露系統的庫存（45 天達到全面覆蓋）。生成真實 SBOM。積極關閉不需要的功能。你無法修補、分割或防禦你不知道存在的東西。 | Risk Control | 90天 |

### HIGH——6個月內

| # | 行動 | 類別 | 時限 |
|---|------|------|------|
| PA 8 | **強化環境**——實施出口過濾（曾阻擋所有公開 Log4j 漏洞利用）。執行深度分割和 Zero Trust。為所有特權帳號強制使用抗網路釣魚 MFA。每個邊界都增加攻擊者成本。 | Risk Control | 6個月 |
| PA 9 | **建立欺騙能力**——欺騙與攻擊工具和漏洞無關，基於 TTP 識別攻擊和攻擊者。部署 canaries 和 honey tokens、分層行為監控、預授權 containment 行動。 | Risk Control | 6個月 |
| PA 11 | **建立 VulnOps**——長期而言，建立一個像 DevOps 一樣運作的永久漏洞運營功能。擁有對整個軟體資產（從自身程式碼到第三方軟體）持續發現零日的能力，並建立自動化修補管道。從一開始就圍繞分級紀律設計。 | Risk Control | 6個月 |

### HIGH——12個月內

| # | 行動 | 類別 | 時限 |
|---|------|------|------|
| PA 10 | **建立自動化回應能力**——改善檢測工程和事件回應能力，使其系統化並盡可能自動化。資產和用戶行為分析、預授權 containment 行動，以及機器速度執行的回應 playbook。 | Risk Control | 12個月 |

## 嚴重性定義

- **CRITICAL（嚴重）**：若未處理將立即暴露
- **HIGH（高）**：45 天內將有重大暴露或風險增加
- **MEDIUM（中）**：有意義的風險增加，若未處理會削弱更高優先順序控制的有效性

## 風險類型定義

- **Threat（威脅）**：外部行動者能力控制會增加成本但無法消除
- **Vulnerability（漏洞）**：可透過補救處理的內部可利用狀況
- **Capability Gap（能力缺口）**：防禦功能缺失或低於要求水平運作
- **Governance（治理）**：放大所有其他風險的組織或結構性故障

## 第一波，未來還有更多

Mythos 只是未來 AI 技術顛覆的第一波。在建立 Mythos-ready 計畫時，我們不僅要尋求回歸平衡，還要為未來浪潮保持平衡做準備。

## 人力成本

安全從業人員正面臨文化挑戰。AI 同時加速了漏洞報告的頻率、組織正在發布的程式碼數量，以及不斷擴大的攻擊面。已經滿負荷的員工還必須應付將 AI 整合到自身工作流程中的認知強度——通常沒有相應的自動化投資、工具、人力或福祉支持。

領導者必須清醒地看待人力成本：倦怠和損耗代表直接的操作風險。駕馭這次轉型所需的專業知識很稀缺，需要多年才能培養，無法在短期內替補。
