---
title: LLM Wiki — 第二個大腦
type: source
topic: llm-wiki
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [方法論, 知識管理, LLM]
sources: []
related: [python-engineering]
summary: 提出「LLM Wiki」模式——由 LLM 增量維護的個人知識庫，區分 raw sources（不可變）和 wiki（可更新），讓知識像代碼一樣複利成長。
---

# LLM Wiki — 第二個大腦

## 核心概念

LLM Wiki 是一種個人知識庫的建構模式，與傳統 RAG 不同：

| 傳統 RAG | LLM Wiki |
|----------|----------|
| 每次查詢從零發現知識 | 知識已編譯、交叉引用 |
| 無累積性 | 持續複利 |
| 矛盾未標記 | 矛盾已被標註 |

## 三層架構

1. **Raw Sources** — 原始資料庫，不可變
2. **Wiki** — LLM 生成的知識網絡，可更新
3. **Schema** — LLM 的作業指導書

## 關鍵 insight

> 維護知識庫最費力的不是閱讀或思考，而是 bookkeeping——更新交叉引用、保持摘要最新、標記矛盾。人類放棄 wiki 是因為維護成本增長快於價值。LLM 不會無聊、不會忘記更新交叉引用、一次能處理 15 個檔案。

## 應用場景

- 個人：目標、健康、心理成長
- 研究：深度論文閱讀，數週到數月
- 閱讀書籍：建立角色、主題、情節的維基
- 團隊：內部 wiki 由 LLM 維護
- 競爭分析、盡職調查、行程規劃

## 操作

- **Ingest**：新來源 → 討論 → 更新 wiki
- **Query**：查詢 wiki → 綜合回答 → 好的回答存回 wiki
- **Lint**：健康檢查（矛盾、過時、孤島、缺失）

## 工具鏈

- Obsidian（IDE）
- Obsidian Web Clipper（擷取網頁）
- Marp（投影片）
- Dataview（查詢 frontmatter）
- qmd（本地搜尋引擎）

## 與 Vannevar Bush's Memex 的關聯

 Bush's 1945 年的 Memex 願景是「個人、精心策劃的知識庫，文件間有 associative trails」。他無法解決的是誰來做維護工作。LLM 解決了這個問題。

---

## See also

- [CLAUDE.md](../../CLAUDE.md) — 本 wiki 的作業指導書（schema）
