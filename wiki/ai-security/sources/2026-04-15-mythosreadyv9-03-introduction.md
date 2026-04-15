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

# III. Introduction（導論）

## 新常態

我們對 AI 在漏洞研究、利用和自主攻擊方面能力的假設可能已經過時。從 2025 年到 2026 年，我們看到了能力持續增強的案例，包括研究環境中和實際野生攻擊中。

AI 驅動的漏洞發現與利用已加速超過一年。詳見附錄 A 的更多歷史證據。

Anthropic 的 Claude Mythos (Preview) 代表了這條軌跡上的階躍變化：自主發現橫跨所有主要作業系統和瀏覽器的數千個關鍵漏洞、在沒有人類指導的情況下生成可用漏洞利用，並賦能自主攻擊編排——速度與規模都超越任何過往能力。

## 結構性不對稱

這個不對稱是結構性的。AI 降低了發現和利用漏洞的成本和技能門檻，速度超過了組織修補它們的能力。從漏洞存在到武器化的視窗已壓縮到小時級。攻擊者獲得不成比例的優勢，而現有的修補週期、回應流程和風險指標並非為這種環境而建構。

## 是什麼讓 Mythos 與眾不同

雖然許多這些能力在這個新模型之前就已經存在，但 Mythos 等級的能力確實代表了階躍變化，並將迅速普及。表現良好的組織將是那些現在就建立肌肉的組織：流程、工具，以及願意將 AI 作為安全核心的文化。

Mythos 展現了三項使其與眾不同的能力：

**複雜的鏈式漏洞**：Mythos 識別由多個原始漏洞鏈接組成的漏洞，例如需要多個記憶體損壞漏洞组合成單一利用路徑的場景。

**「一擊」（單一 prompt）能力**：Mythos 用單一 prompt 完成比以往多很多的工作，無需複雜的 scaffolding 或 Agent 設定。

## Mythos 與 Glasswing：為何重要

### Mythos

Mythos 在技術和策略層面上都與先前的能力不同。技術上：

**無需 scaffolding 的漏洞利用**：Anthropic 內部實驗室環境測試顯示，Mythos 在 Firefox 上生成了 **181 個可用漏洞利用**，而 Claude Opus 4.6 在同樣條件下僅成功 2 次，標誌著自主性和可靠性的實質躍升。

### Project Glasswing

Mythos 的規模和速度促使 Anthropic 創建了 Project Glasswing，這可能是史上最大規模的多方漏洞協調行動。Anthropic 向選定的關鍵基礎設施供應商、產業夥伴和開源維護者提供了 Mythos 的早期訪問，讓他們能夠修補自己的產品。

**Project Glasswing 最主要的限制**是它只能覆蓋這麼多。全球可利用的攻擊面遠遠大於任何精心策劃的合作夥伴生態系統所能覆蓋的範圍。

雖然 Glasswing 建立的協調模型非常重要，但其影響將嚴重取決於它能多快地擴大覆蓋範圍，以及修補和披露管道是否能跟上 AI 進展和對手採用的步伐。

## LLM 攻擊能力演化時間線（2025-2026）

| 日期 | 事件 |
|------|-------|
| 2025年6月24日 | XBOW 登上 HackerOne 排行榜首——首個在平台上超越所有人類駭客的自主系統 |
| 2025年8月5日 | Google Big Sleep 在 FFmpeg、ImageMagick 中發現 20 個真實零日 |
| 2025年8月8日 | DARPA AIxCC 決賽在 DEF CON 33——4 小時內在 5400 萬行程式碼中發現 54 個漏洞 |
| 2025年9月 | Adkins 與 Evron 發布 Singularity 警告——自主漏洞發現約落後 6 個月 |
| 2025年11月14日 | 首個 AI 編排的間諜活動——中國國家級組織使用 Claude Code 在約 30 個全球目標上執行完整攻擊鏈 |
| 2026年2月5日 | Anthropic 報告在開源軟體中發現 500+ 個高危漏洞 |
| 2026年2月5日 | AISLE 發現 12 個 OpenSSL 零日，包括一個可追溯至 1998 年的 CVSS 9.8 漏洞 |
| 2026年3月 | Linux kernel 漏洞報告從每週 2 個攀升至 10 個（最初是幻覺，現在全部驗證為真實漏洞） |
| 2026年3月 | Sysdig 記錄 AI 攻擊在 8 分鐘內取得 admin 權限 |
| 2026年3月 | [un]prompted 會議 + Zero Day Clock 發布——時間到利用現在不到一天 |
| 2026年4月7日 | Claude Mythos Preview + Project Glasswing 啟動——發現橫跨所有主要 OS 和瀏覽器的數千個零日、72% 漏洞成功率、發現 27 年歷史的 OpenBSD 漏洞 |
