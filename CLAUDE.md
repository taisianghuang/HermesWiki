# LLM Wiki — 第二個大腦

## 概述

這是一個由 LLM 增量維護的個人知識庫。核心原則：

- **Raw sources** 是你的原始資料庫（不可變）
- **Wiki** 是 LLM 生成的可維護知識網絡（可更新）
- **Schema**（本文件）是 LLM 的作業指導書

每次對話都是一次 wiki 操作：ingest（攝入新來源）、query（查詢知識）、lint（健康檢查）。

---

## 目錄結構

```
~/HermesWiki/
├── CLAUDE.md              # 本文件 — schema
├── index.md               # 頂層入口 — 導向 wiki/_master-index.md
├── log.md                 # 操作日誌 — 按時間順序記錄所有操作
│
├── assets/                # 附件：圖片、圖表、下載的資源
│
├── raw/                   # 原始來源（YOUR inbox — 不可變）
│   ├── hermes-agent/      #   Hermes Agent repo 原始檔
│   └── ...                #   任何新來源都可以直接丟進來
│
├── wiki/                  # LLM 維護的知識網絡（LLM's domain）
│   ├── _master-index.md   #   頂層總索引
│   ├── hermes-agent/      #   主題：Hermes Agent 框架
│   │   ├── _index.md     #   主題局部索引
│   │   ├── concepts/     #   概念頁面
│   │   └── sources/       #   來源摘要頁
│   ├── python-engineering/ # 主題：Python 工程化
│   │   ├── _index.md
│   │   ├── concepts/
│   │   └── sources/
│   ├── llm-wiki/          #   主題：Wiki 系統本身
│   │   ├── _index.md
│   │   └── sources/
│   └── (未來新增主題...)   #   新主題直接新增資料夾
│
└── output/                # 查詢結果與報告（消耗品）
    ├── query-results.md   #   知識查詢的產出
    └── slide-decks/       #   簡報輸出
```

---

## 核心差異：扁平 vs 階層

| 舊架構（扁平） | 新架構（階層） |
|--------------|--------------|
| `wiki/concepts/` 所有概念放一資料夾 | 每個主題有自己的 `concepts/` 和 `sources/` |
| `wiki/sources/` 所有來源放一資料夾 | 主題隔離，cross-reference 更清晰 |
| `wiki/entities/`、`analyses/` 空白 | 改為每主題自己管理，未來彈性更大 |
| 單一 `index.md` | 雙層索引：`_master-index.md` + 各主題 `_index.md` |
| 無 output 概念 | `output/` 分離消耗品，不污染 wiki |

---

## 頁面命名與格式

### 檔案名稱
- 使用 kebab-case：`machine-learning-llms.md`
- 包含日期區分：`2026-04-13-ai-trends.md`
- 來源頁面：與 raw 檔名對應

### Frontmatter（每頁必備）
```yaml
---
title: 頁面標題
type: concept | entity | source | analysis
created: 2026-04-13
updated: 2026-04-13
created_by: hermes | tai | claude code  # 必填！誰創建了這個頁面
tags: [tag1, tag2]
sources: []  # 引用哪些 raw 來源（用相對路徑）
related: []  # 相關 wiki 頁面（用標題）
summary: 一句話描述（用於 _index.md）
---
```

### 頁面結構
1. **標題**（H1）
2. **Frontmatter**
3. **Summary**（簡短摘要，2-3 句）
4. **Content**（正文）
5. **See also**（相關連結）

---

## 操作流程

### 1. Ingest（新來源攝入）

**觸發**：用戶提供新來源（檔案、連結、貼上文字）

**流程**：
1. 讀取來源內容，與用戶討論要點（必要時）
2. 在 `raw/` 建立原始檔案
3. 判斷所屬主題（有現有主題 → 放入該主題；新主題 → 新增資料夾）
4. 在該主題的 `sources/` 建立摘要頁
5. 更新該主題的 `_index.md`
6. 更新 `wiki/_master-index.md`
7. 更新相關的 concept 頁面（新增或修訂）
8. 在 `log.md` 記錄 ingest

**新主題判斷**：
- `hermes-agent`：Hermes Agent 框架相關
- `python-engineering`：Python 工程化、DevOps、工具鏈
- `llm-wiki`：Wiki 系統本身的概念與設計
- 其他：自由新增（如 `web-development`、`crypto-research` 等）

**原則**：
- 單一來源可能影響 10-15 個 wiki 頁面
- 新資訊挑戰舊 claim 時，在舊頁面標註「已被取代」並註明新來源
- 保持 cross-reference 一致性

### 2. Query（知識查詢）

**觸發**：用戶提問

**流程**：
1. 讀取 `wiki/_master-index.md` 找到相關主題
2. 讀取該主題的 `_index.md` 找到相關頁面
3. 讀取相關頁面
4. 綜合回答，引用（`[來源標題](路徑)`）
5. **重要**：如果回答有價值，主動詢問是否存入該主題的 `analyses/` 或 `output/`

**回答形式**：
- 短問答：直接回覆
- 分析：詳細回覆
- 有價值的發現：存入 `wiki/<topic>/analyses/` 或 `output/`

### 3. Lint（健康檢查）

**觸發**：用戶要求 `/lint` 或每週例行檢查

**檢查項目**：
- 矛盾：跨頁面的聲明是否衝突
- 過時：新來源是否已取代舊內容
- 孤島：沒有 inbound link 的頁面
- 缺失：重要概念有提到但無專頁
- 缺口：哪些重要問題還未研究
- 漂移：頁面內容偏離所屬主題

**輸出**：發現清單 + 修復建議

---

## _master-index.md 格式

```markdown
# HermesWiki — Master Index

## Topics（主題目錄）

| 主題 | 說明 | 頁面數 |
|------|------|--------|
| [hermes-agent/](hermes-agent/_index.md) | ... | 25 |
| [python-engineering/](python-engineering/_index.md) | ... | 32 |

## Quick Links
- [raw/](../raw/) — 原始來源
- [output/](../output/) — 查詢結果與報告
- [log.md](../log.md) — 操作日誌
```

---

## <topic>/_index.md 格式

```markdown
# topic-name

> 主題簡短描述

## Concepts

| 頁面 | 摘要 | 更新 |
|------|------|------|
| [頁面](concepts/page.md) | 一句話 | YYYY-MM-DD |

## Sources

| 頁面 | 摘要 | 更新 |
|------|------|------|
| [頁面](sources/page.md) | 一句話 | YYYY-MM-DD |

## Statistics
- **Concepts**: N
- **Sources**: N
- **Last updated**: YYYY-MM-DD
```

---

## log.md 格式

```markdown
# Log

## [YYYY-MM-DD] ingest | 來源標題
- 主題：topic-name
- 處理的頁面：concept1, source1
- 關鍵發現：...

## [YYYY-MM-DD] query | 用戶問題簡述
- 回答來源：page1, page2
- 是否存入 output/：否

## [YYYY-MM-DD] lint
- 發現：問題1, 問題2
- 已修復：...
```

---

## Wiki 維護原則

1. **Consistency**：更新任何頁面時，檢查 cross-reference 是否需要一併更新
2. **Attribution**：所有資訊必須標註來源
3. **Traceability**：log.md 記錄所有操作
4. **Provenance**：raw/ 永不修改
5. **Incremental**：每次操作都讓 wiki 變得更豐富
6. **Topic Isolation**：不同主題的頁面放在各自主題資料夾，不要混在一起

---

## 提示

- Obsidian Web Clipper 可以快速把網頁存入 raw/
- 圖片下載到 assets/，LLM 可直接查看
- 好的分析頁面是 wiki 價值的關鍵 — 你的發現和洞見值得保留
- Git commit 你的 wiki，享受版本控制
- output/ 是消耗品（query results、reports），不需要 version control
