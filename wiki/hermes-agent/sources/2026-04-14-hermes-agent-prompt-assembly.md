---
title: Hermes Prompt Assembly — Official Doc
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, prompt, official-docs]
sources: [../../raw/hermes-agent/docs/prompt-assembly.md]
related: [hermes-prompt-assembly, hermes-memory-system, hermes-skills-system]
summary: 官方 Prompt Assembly 文檔，詳細說明 10 層 cached system prompt 組裝順序、SOUL.md 邏輯、Context Files priority system（first match wins）、API-call-time-only layers 的分離原因。
---

# Hermes Prompt Assembly — Official Doc

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | https://hermes-agent.nousresearch.com/docs/developer-guide/prompt-assembly |
| **Raw 位置** | `raw/hermes-agent/docs/prompt-assembly.md` |
| **涵蓋內容** | 10 層 prompt 組裝、SOUL.md、Context Files、Skills index、API-call-time layers |

## 與既有概念的差異

| 項目 | 既有概念頁 | 官方文檔 |
|------|----------|---------|
| **層數** | 10 層（我有）| 10 層（一致）|
| **SOUL.md** | 有描述 | 更詳細的 loading logic + skip_soul 機制 |
| **Context Files** | Priority table | 完整 priority system + first match wins 邏輯 |
| **API-call-time layers** | 列舉 5 種 | 更詳細的 separation rationale |

## 官方文檔原始 content

（完整內容收錄於 `raw/hermes-agent/docs/prompt-assembly.md`）
