---
title: Hermes Environments — Atropos RL Training Integration
type: source
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, rl-training, atropos, benchmark, environments, swe-bench]
sources: [../../raw/hermes-agent/environments/README.md]
related: [hermes-agent-core-loop, hermes-tool-registry]
summary: Hermes Agent 與 Atropos RL 訓練框架的整合，涵蓋三層繼承架構（BaseEnv → HermesAgentBaseEnv → ConcreteEnvs）、ToolContext、Phase 1/2 雙階段訓練、12 種 tool call parsers。
---

# Hermes Environments — Atropos RL Training Integration

## 基本資訊

| 欄位 | 內容 |
|------|------|
| **來源** | `raw/hermes-agent/environments/README.md` |
| **涵蓋內容** | Atropos 整合、三層繼承架構、ToolContext、Phase 1/2 雙階段訓練、tool call parsers、具體環境 |

## 為什麼需要 Environments？

Hermes Agent 不只是對話系統，還是 **RL 訓練引擎**。可用來：
- 訓練程式碼任務（SWE-bench style）
- 評估 benchmark（Terminal-Bench 2.0）
- 生成 SFT 訓練資料

## 三層繼承架構

```
Atropos BaseEnv (atroposlib)
  - Server management、VLLM/SGLang Worker scheduling、Wandb logging
         ↑ inherits
HermesAgentBaseEnv (hermes_base_env.py)
  - Terminal backend、Tool resolution、Agent loop、ToolContext、Async patches
         ↑ inherits
TerminalTestEnv / HermesSweEnv / TerminalBench2EvalEnv
```

### HermesAgentBaseEnv 職責

- 設定 `TERMINAL_ENV` 環境變數（決定要用哪種 terminal backend）
- 解析 hermes-agent toolsets（透過 `get_tool_definitions()`）
- 實作 `collect_trajectory()`：執行完整 agent loop 並計算 rewards
- 支援 Phase 1 / Phase 2 雙階段操作

## ToolContext（per-rollout 工具存取）

每個 rollout 有一個 `ToolContext`，讓 reward function 直接存取所有 hermes-agent 工具，且狀態（檔案、程序、瀏覽器分頁）與模型操作的是**同一個** session：

```python
async def compute_reward(self, item, result, ctx: ToolContext):
    test = ctx.terminal("pytest -v")
    if test["exit_code"] == 0:
        return 1.0

    content = ctx.read_file("/workspace/solution.py")
    if content.get("content"):
        return 0.5

    ctx.download_file("/remote/output.bin", "/local/output.bin")
```

可用方法：`terminal()`、`read_file()`/`write_file()`/`search()`、`upload_file()`/`download_file()`、`web_search()`/`web_extract()`、`browser_navigate()`/`browser_snapshot()`、`call_tool()`（任意工具）。

## 雙階段訓練

### Phase 1：OpenAI Server（Evaluation / SFT）

使用 `server.chat_completion()` 的 `tools=` 參數，server 原生處理 tool call parsing。適合 evaluation、SFT data generation。

### Phase 2：VLLM ManagedServer（Full RL Training）

透過 `/generate` 端點取得精確 token IDs + logprobs，用戶端 `tool_call_parsers/` 從 raw 輸出重建 structured `tool_calls`。支援 GRPO/PPO 完整 RL 訓練。

## Tool Call Parsers（12 種格式）

Phase 2 需要的 client-side parsers：

| Parser | 模型 |
|--------|------|
| `hermes` | Hermes/ChatML XML |
| `mistral` | Mistral `[TOOL_CALLS]` |
| `llama3_json` | Llama 3 JSON |
| `qwen` / `qwen3_coder` | Qwen / Qwen3 Coder |
| `deepseek_v3` / `deepseek_v3_1` | DeepSeek V3 / V3.1 |
| `kimi_k2` | Kimi K2 |
| `longcat` | Longcat |
| `glm45` / `glm47` | GLM model formats |

```python
from environments.tool_call_parsers import get_parser
parser = get_parser("hermes")
content, tool_calls = parser.parse(raw_model_output)
```

## 具體環境

| 環境 | 用途 | 模式 |
|------|------|------|
| `TerminalTestEnv` | 端到端棧驗證（內建任務，無需外部 dataset） | serve / process |
| `HermesSweEnv` | SWE-bench 風格訓練（Modal sandbox） | serve |
| `TerminalBench2EvalEnv` | Terminal-Bench 2.0 評估（89 任務，Eval-only） | evaluate |
| `tblite` | 100 個校準任務（TB2 快速 proxy） | evaluate |
| `yc_bench` | 長視野策略 benchmark | evaluate |

## 與 Core Agent Loop 的差異

| 項目 | AIAgent（對話） | HermesAgentLoop（訓練） |
|------|----------------|----------------------|
| 用途 | 互動式對話 | RL rollout |
| 獎勵 | 無 | `compute_reward()` 函式 |
| 終端後端 | 可插拔 | 可插拔 |
| 執行次數 | 單次 | 平行 rollout 多次 |
| 輸出 | 回應文字 | `AgentResult`（對話歷史、turn 數、reasoning、tool errors）|

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Tool Registry](hermes-tool-registry.md)
