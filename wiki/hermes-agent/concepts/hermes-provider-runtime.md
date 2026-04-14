---
title: Hermes Provider Runtime Resolution
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, provider, api-key, credentials, models]
sources: [../../raw/hermes-agent/docs/architecture.md, ../../raw/hermes-agent/docs/provider-runtime.md]
related: [hermes-agent-core-loop, hermes-prompt-assembly]
summary: Hermes 的 Provider Runtime Resolution 是跨 CLI/Gateway/Cron/ACP/Auxiliary 的共享系統。Resolution precedence 為：explicit CLI arg > config.yaml > env vars > defaults。支援 18+ provider families，包括 AI Gateway、OpenRouter、Anthropic、DeepSeek、MiniMax 等。
---

# Hermes Provider Runtime Resolution

## 概述

Hermes 的 Provider Runtime Resolution 是跨多個進入點共享的系統：

- CLI
- Gateway
- Cron jobs
- ACP
- Auxiliary model calls

Primary implementations:
- `hermes_cli/runtime_provider.py` — credential resolution, `_resolve_custom_runtime()`
- `hermes_cli/auth.py` — provider registry, `resolve_provider()`
- `hermes_cli/model_switch.py` — shared `/model` switch pipeline（CLI + gateway 共用）
- `agent/auxiliary_client.py` — auxiliary model routing

## Resolution Precedence

```
1. Explicit CLI/runtime request      ← 最高優先
2. config.yaml model/provider config
3. Environment variables
4. Provider-specific defaults or auto resolution
```

這個順序很重要，因為 Hermes 把用戶保存的 model/provider choice 當作 source of truth，防止 stale shell export 默默覆寫用戶在 `hermes model` 最後選擇的 endpoint。

## Provider Families（18+）

| Provider | Description |
|----------|-------------|
| AI Gateway (Vercel) | Vercel AI Gateway |
| OpenRouter | OpenRouter |
| Nous Portal | Nous Research portal |
| OpenAI Codex | OpenAI Codex / Responses API |
| Copilot / Copilot ACP | GitHub Copilot |
| Anthropic (native) | Native Anthropic Messages API |
| Google / Gemini | Google AI / Gemini |
| Alibaba / DashScope | Alibaba Cloud |
| DeepSeek | DeepSeek |
| Z.AI | Z.AI |
| MiniMax | MiniMax |
| ... | 還有更多 |

## API Mode Resolution

Provider 決定 API mode：

```
anthropic provider → anthropic_messages
OpenAI-compatible → chat_completions
Codex → codex_responses
```

## Credential Resolution

每個 provider 的 credential resolution 流程：
1. 檢查 `HERMES_API_KEY` 或 provider-specific env var
2. 檢查 `~/.hermes/.env` 中的 API key
3. 檢查 `config.yaml` 中的自定義 endpoint

## Auxiliary Model Routing

Auxiliary tasks（vision、compression、web extraction、session search）有獨立的 fallback chain：

```yaml
auxiliary:
  fallback_providers:
    vision: ["anthropic", "openai"]
    compression: ["anthropic", "openrouter"]
```

## Fallback Model（主要模型）

當 primary model 失敗時：
- 429 rate limit
- 5xx server error
- 401/403 auth error

系統會檢查 `fallback_providers` list，按順序嘗試。如果遇到 401/403，會先嘗試刷新 credential 再 failover。

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Prompt Assembly](hermes-prompt-assembly.md)
