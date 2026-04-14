---
title: Hermes Context Compression — Official Doc
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, context-compression, official-docs]
sources: [../../raw/hermes-agent/docs/context-compression-and-caching.md]
related: [hermes-context-compression, hermes-agent-core-loop]
summary: 官方 Context Compression and Caching 文檔，涵蓋 Pluggable Context Engine、雙層壓縮系統（85%/50%）、4 階段 compression algorithm、Anthropic Prompt Caching（system_and_3 策略）。
---

# Hermes Context Compression — Official Doc

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | https://hermes-agent.nousresearch.com/docs/developer-guide/context-compression-and-caching |
| **Raw 位置** | `raw/hermes-agent/docs/context-compression-and-caching.md` |
| **涵蓋內容** | Pluggable Context Engine、雙層壓縮、4 階段算法、Anthropic Caching、Context Pressure Warnings |

## 與既有概念頁的差異

| 項目 | 既有概念頁 | 官方文檔 |
|------|----------|---------|
| **Pluggable Engine** | 有 | 更詳細的 resolution order + plugin 機制 |
| **雙層 threshold** | 85%/50% | 詳細說明為何 gateway 是 85%（不是 50%）|
| **Phase 1** | 有 | 更詳細：prune >200 chars tool results |
| **Boundary Alignment** | 無 | 有 `_align_boundary_backward()` 詳細說明 |
| **Before/After Example** | 無 | 有完整 45→25 messages 範例 |
| **Context Pressure** | 有 | 詳細說明 warning 邏輯 |

## 官方文檔原始 content

（完整內容收錄於 `raw/hermes-agent/docs/context-compression-and-caching.md`）
