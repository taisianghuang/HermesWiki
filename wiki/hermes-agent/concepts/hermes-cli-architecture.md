---
title: Hermes CLI Architecture
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, cli, architecture, slash-commands]
sources: [../sources/2026-04-14-hermes-agent-repo.md]
related: [hermes-agent-core-loop, hermes-tool-registry, hermes-gateway]
summary: Hermes CLI 是用 Rich + prompt_toolkit 建構的互動式終端介面，透過 CommandDef 集中定義所有斜線指令，Skin Engine 支援資料驅動的主題切換。
---

# Hermes CLI Architecture

## 概述

Hermes CLI 是 Hermes 的**互動式終端使用者介面**，使用 Rich（UI）和 prompt_toolkit（輸入）建構，支援多行編輯、斜線指令自動完成、對話歷史和 interrupt 重新導向。

## 核心技術棧

| 函式庫 | 用途 |
|--------|------|
| **Rich** | Banner、Panels、彩色輸出 |
| **prompt_toolkit** | 輸入框、自動完成、多行編輯 |
| **KawaiiSpinner** (`agent/display.py`) | API 呼叫時的可愛動畫 |

## 架構結構

```
cli.py  (~447KB, ~12000+ lines)
    │
    ├── load_cli_config()     # 合併設定檔 + YAML
    ├── HermesCLI class
    │   ├── process_command()  # 指令分發
    │   ├── chat_loop()        # 對話迴圈
    │   └── ...
    │
    └── hermes_cli/
        ├── commands.py    # CommandDef 斜線指令定義
        ├── skin_engine.py # 主題引擎
        ├── config.py      # 預設設定、遷移
        └── ...
```

## 斜線指令系統（Slash Commands）

### 集中定義的好處

所有斜線指令在 `hermes_cli/commands.py` 的 `COMMAND_REGISTRY` 集中定義：

```python
CommandDef("mycommand", "Description of what it does", "Session",
           aliases=("mc",), args_hint="[arg]")
```

這個定義自動影響：
- CLI `/help` 輸出
- Gateway `/help` 輸出
- Telegram BotCommand menu
- Slack subcommand routing
- Autocomplete

### 新增斜線指令

**步驟 1**：在 `hermes_cli/commands.py` 加入 `CommandDef`：

```python
CommandDef("mycommand", "Description", "Session",
           aliases=("mc",), args_hint="[arg]")
```

**步驟 2**：在 `cli.py` 的 `process_command()` 加入 handler：

```python
elif canonical == "mycommand":
    self._handle_mycommand(cmd_original)
```

**步驟 3**：如果需要在 Gateway 使用，在 `gateway/run.py` 加入 handler：

```python
if canonical == "mycommand":
    return await self._handle_mycommand(event)
```

## 指令類別（Command Categories）

| Category | 說明 | 範例 |
|----------|------|------|
| Session | 對話控制 | /new, /reset, /retry |
| Configuration | 設定調整 | /model, /personality |
| Tools & Skills | 工具與技能 | /skills, /tools |
| Info | 資訊查詢 | /help, /usage |
| Exit | 離開 | /exit |

## Skin Engine（主題引擎）

`hermes_cli/skin_engine.py` 提供了**資料驅動**的 CLI 主題系統。

### 內建主題

| 主題 | 特色 |
|------|------|
| `default` | 經典 Hermes 金色/kawaii 風格 |
| `ares` | 深紅/青銅戰神主題 |
| `mono` | 簡潔灰階單色 |
| `slate` | 冷色調開發者主題 |

### 可自訂項目

- Banner 邊框/標題/強調/啞光/正文顏色
- Spinner 表情（等待中/思考中）
- Spinner 動詞（thinking verbs）
- Spinner 翅膀（可選）
- 工具前綴（`┊`）
- 回應框標籤
- Agent 名稱

### 使用方式

```bash
/hermes skin ares    # 切換主題
/hermes skin list    # 列出可用主題
```

## KawaiiSpinner

`agent/display.py` 中的 `KawaiiSpinner` 類：

- 在 API 呼叫時顯示可愛表情動畫
- 支援 Thinking mode（顯示思考中表情）
- 工具結果使用 `┊` 前綴逐行輸出

## 設定檔系統

Hermes 使用兩套設定系統：

| 載入器 | 使用場景 | 位置 |
|--------|---------|------|
| `load_cli_config()` | CLI 模式 | `cli.py` |
| `load_config()` | `hermes tools`, `hermes setup` | `hermes_cli/config.py` |
| 直接 YAML load | Gateway | `gateway/run.py` |

## 主要 CLI 命令

```bash
hermes              # 啟動互動式 CLI
hermes model        # 選擇 LLM provider/model
hermes tools        # 配置啟用的工具
hermes config set   # 設定個別配置值
hermes gateway      # 啟動訊息 Gateway
hermes setup        # 執行完整設定精靈
hermes update       # 更新到最新版本
hermes doctor        # 診斷問題
```

## 與 Gateway 的共享機制

CLI 和 Gateway 共享：
- 同一個 `COMMAND_REGISTRY`
- 同一個 `resolve_command()` 解析器
- 同一套斜線指令 handler

差異：
- CLI 有 `KawaiiSpinner` 和 Rich UI
- Gateway 有 Platform Adapter

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Tool Registry](hermes-tool-registry.md)
- [Hermes Gateway](hermes-gateway.md)
