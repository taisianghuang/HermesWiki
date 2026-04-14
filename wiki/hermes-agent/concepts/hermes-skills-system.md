---
title: Hermes Skills System
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, skills, memory, learning-loop]
sources: [../sources/2026-04-14-hermes-agent-repo.md]
related: [hermes-agent-core-loop, hermes-memory-system]
summary: Hermes Skills 是 Agent 的「程序記憶」系統，允許 Agent 從經驗中創建和改進技能（.md 文件），技能在下次相同場景中自動被调用，形成自我改進的學習循環。
---

# Hermes Skills System

## 概述

Hermes 的 Skills 系統是它的**自我改進能力核心**。當 Agent 完成複雜任務後，它會自動創建或更新一個 Skill 文件（Markdown 格式），紀錄「如何完成這件事」。下次遇到類似場景時，Skill 會被自動調用，形成一個**閉環學習系統**。

## 關鍵特性

1. **自動創建**：完成複雜任務後自動生成
2. **自我改進**：使用過程中會不斷優化
3. **持久化**：技能以 .md 文件形式存在 `~/.hermes/skills/`
4. **自動調用**：遇到類似場景時自動被撈出來用
5. **跨 session**：不同對話之間共享

## 目錄結構

```
~/.hermes/skills/
├── SKILL.md          # 技能定義檔（每個技能一個資料夾）
├── references/       # 參考文檔
├── scripts/          # 輔助腳本
├── templates/        # 模板
└── assets/          # 資源
```

## Skill 文件格式

```markdown
---
name: my-skill
description: 描述這個技能是做什麼的
version: 1.0.0
category: devops
tags: [docker, deployment, ci-cd]
trigger: 什麼條件會觸發這個技能
---

# My Skill

## 觸發條件
- 什麼情況下應該使用這個技能

## 使用步驟
1. 步驟一
2. 步驟二

## 範例
```bash
example command
```

## 陷阱與注意事項
- 這個技能有哪些坑
```

## Skills Hub

Hermes 支援從 [agentskills.io](https://agentskills.io) 安裝社群共享的技能：

```
hermes skills install <skill-name>   # 安裝技能
hermes skills list                   # 列出已安裝技能
hermes skills update                 # 更新技能
```

## Skills 與 Tool 的區別

| 特性 | Skill | Tool |
|------|-------|------|
| 本質 | 一組操作指引（文件） | 可執行的函式 |
| 創建方式 | Agent 自動生成或手動創建 | 程式碼定義 |
| 觸發方式 | 自動（遇到類似場景） | Agent 自行判斷呼叫 |
| 靈活性 | 高（可包含步驟、範例、陷阱） | 中（受限於函式簽名） |
| 存储位置 | `~/.hermes/skills/` | `tools/` 目錄 |

## Skills 與 Memory 的區別

| 特性 | Skill | Memory |
|------|-------|--------|
| 內容 | 操作程序/指引 | 領域事實/偏好 |
| 創建者 | Agent 自己（從經驗）| Agent 或用戶 |
| 使用方式 | 自動被 prompt builder 注入 | 需要時查詢 |
| 顆粒度 | 整個技能文件 | 片段記憶 |

## Prompt 中的 Skills

Skills 透過 `agent/skill_commands.py` 處理：

```python
# skill_commands.py 掃描 ~/.hermes/skills/
# 技能被注入為 user message（而非 system prompt）
# 這樣可以保留 prompt caching 的優勢
```

## 內建 Skills

Hermes 自帶一些預設 Skills：
- `brainstorming` — 創意發想流程
- `code-review` — 程式碼審查
- `systematic-debugging` — 系統化除錯
- `test-driven-development` — TDD 流程

## 學習循環

```
任務完成
    │
    ▼
Hermes 檢測複雜度
    │
    ├── 簡單任務 → 直接回覆
    │
    └── 複雜任務 → 自動創建/更新 Skill
                       │
                       ▼
                  下次遇到類似任務
                       │
                       ▼
                  Skill 被自動調用
                       │
                       ▼
                  任務完成得更快/更好
```

## 與 agentskills.io 的關係

Hermes Skills 系統相容於 [agentskills.io](https://agentskills.io) 開放標準，可以：
- 從 agentskills.io 安裝社群技能
- 發布自己的技能到 agentskills.io
- 技能定義格式與其他 Agent 框架互通

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Memory System](hermes-memory-system.md)
