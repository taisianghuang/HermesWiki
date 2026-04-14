---
title: Hermes Environments — RL Training Integration
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, rl-training, atropos, benchmark, environments, swe-bench]
sources: [../../raw/hermes-agent/environments/README.md, ../sources/2026-04-14-hermes-environments-rl.md]
related: [hermes-agent-core-loop, hermes-tool-registry]
summary: Hermes 將 Agent 能力延伸到 RL 訓練，透過 Atropos 整合提供 SWE 訓練、SWE-bench 風格評估、Terminal-Bench 2.0 benchmark，支援 Phase 1（Evaluation/SFT）和 Phase 2（Full RL）雙階段。
---

# Hermes Environments — RL Training Integration

## 概述

Hermes Agent 不只是對話系統，還是 **RL 訓練引擎**。透過與 Atropos RL 框架的整合，可用來：
- 訓練程式碼任務（SWE-bench style）
- 評估 benchmark（Terminal-Bench 2.0）
- 生成 SFT 訓練資料

## 三層繼承架構

```
Atropos BaseEnv (atroposlib)
  - Server management、VLLM/SGLang、Worker scheduling、Wandb logging
         ↑ inherits
HermesAgentBaseEnv (hermes_base_env.py)
  - Terminal backend、Tool resolution、Agent loop、ToolContext
         ↑ inherits
TerminalTestEnv / HermesSweEnv / TerminalBench2EvalEnv / tblite / yc_bench
```

## ToolContext：Bridge 訓練與執行

每個 rollout 有獨立的 `ToolContext`，讓 reward function 直接存取模型操作過的同一個 session（檔案、程序、瀏覽器狀態）。這樣 reward function 可以：
- 在模型的 sandbox 環境中執行測試
- 檢查模型寫入的檔案內容
- 跨主機傳輸檔案進行驗證

## 雙階段訓練

| 階段 | 用途 | Tool Call Parsing |
|------|------|------------------|
| **Phase 1** | Evaluation / SFT Data Generation | Server 原生處理（OpenAI-compatible `tools=`） |
| **Phase 2** | Full RL Training（GRPO/PPO） | Client-side parser 從 raw 輸出重建 |

Phase 2 需要 12 種 client-side parsers（Hermes、Mistral、Llama3、Qwen、DeepSeek 等）。

## 具體環境對照

| 環境 | 用途 | 模式 | 特點 |
|------|------|------|------|
| `TerminalTestEnv` | 端到端驗證 | serve/process | 內建任務，無需外部 dataset |
| `HermesSweEnv` | SWE-bench 訓練 | serve | Modal sandbox + 測試驗證 |
| `TerminalBench2EvalEnv` | TB2 benchmark | evaluate | 89 個 Docker-based 任務 |
| `tblite` | TB2 proxy | evaluate | 100 個快速校準任務 |
| `yc_bench` | 長視野策略 | evaluate | 戰略性任務 |

## 與 Core Agent Loop 的核心差異

| 維度 | AIAgent（對話） | Environments（訓練） |
|------|----------------|---------------------|
| 目標 | 產生正確回應 | 最大化 reward |
| 回饋機制 | 無 | `compute_reward()` 函式 |
| 執行次數 | 單次互動 | 平行 rollout 多次 |
| 評估方式 | 主觀 | 客觀測試/指標 |
| 典型輸出 | 回應文字 | `AgentResult`（含推理過程、tool calls、錯誤）|

## 延伸閱讀

如果想建立自己的訓練環境，只需：
1. 繼承 `HermesAgentBaseEnv`
2. 實作 `setup()`、`get_next_item()`、`format_prompt()`、`compute_reward()`
3. 用 Atropos CLI 執行 `serve` / `process` / `evaluate`

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Tool Registry](hermes-tool-registry.md)
