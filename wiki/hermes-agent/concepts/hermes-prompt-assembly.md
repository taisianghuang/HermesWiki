---
title: Hermes Prompt Assembly
type: concept
topic: hermes-agent
created: 2026-04-14
updated: 2026-04-14
created_by: hermes
tags: [hermes-agent, prompt, memory, context-files, soul-md]
sources: [../../raw/hermes-agent/docs/architecture.md, ../../raw/hermes-agent/docs/prompt-assembly.md]
related: [hermes-agent-core-loop, hermes-memory-system, hermes-context-compression, hermes-skills-system]
summary: Hermes 的 Prompt 組裝分為「cached system prompt」和「API-call-time additions」兩層。System prompt 由 10 層組成（SOUL.md identity、工具引導、Honcho block、Memory snapshot、USER snapshot、Skills index、Context files、Timestamp、Platform hint），兩層刻意分離以保留 prompt caching 效果。
---

# Hermes Prompt Assembly

## 核心設計原則

Hermes 刻意分離兩種 prompt 狀態：

| 類型 | 位置 | 特性 |
|------|------|------|
| **Cached system prompt** | 持久化，跨 turn | 不會在中途改變 |
| **API-call-time additions** | 暫時性 | ephemeral_system_prompt、prefill、gateway session context overlays |

這個分離是為了：
- 保留 provider-side prompt caching
- 避免不必要的 history mutation
- 保持 memory semantics 可理解
- 讓 gateway/ACP/CLI 可以添加 context 而不影響 persistent prompt state

## Cached System Prompt Layers（10 層順序組裝）

```
Layer 1: Agent Identity (SOUL.md from ~/.hermes/SOUL.md)
Layer 2: Tool-aware behavior guidance (from prompt_builder.py)
Layer 3: Honcho static block (when active)
Layer 4: Optional system message (from config or API)
Layer 5: Frozen MEMORY snapshot
Layer 6: Frozen USER profile snapshot
Layer 7: Skills index
Layer 8: Context files (AGENTS.md, .cursorrules, etc.)
Layer 9: Timestamp / session ID
Layer 10: Platform hint
```

### Layer 1 — Agent Identity (SOUL.md)

`SOUL.md` 位於 `~/.hermes/SOUL.md`，作為 agent 的身份標識，是 system prompt 的第一個 section。

```python
# From agent/prompt_builder.py (simplified)
def load_soul_md() -> Optional[str]:
    soul_path = get_hermes_home() / "SOUL.md"
    if not soul_path.exists():
        return None
    content = soul_path.read_text(encoding="utf-8").strip()
    content = _scan_context_content(content, "SOUL.md")  # Security scan
    content = _truncate_content(content, "SOUL.md")    # Cap at 20k chars
    return content
```

如果 `SOUL.md` 不存在，fallback 到 `DEFAULT_AGENT_IDENTITY`：

```python
DEFAULT_AGENT_IDENTITY = """You are Hermes Agent, an intelligent AI assistant
created by Nous Research. You are helpful, knowledgeable, and direct.
You assist users with a wide range of tasks..."""
```

### Layer 2 — Tool-Aware Behavior Guidance

來自 `prompt_builder.py` 的工具使用指引，包括：
- Memory tool 使用說明
- `session_search` 使用說明
- Tool-use enforcement（GPT/Codex models 專用）

### Layer 5 & 6 — Memory Snapshots

- **MEMORY snapshot**：持久化記憶（`~/.hermes/memory/MEMORY.md`）
- **USER snapshot**：用戶 profile（`~/.hermes/memory/USER.md`）

Mid-session writes 會更新磁碟狀態，但**不會**改變已經構建的 system prompt，直到新 session 或強迫 rebuild。

### Layer 8 — Context Files（專案上下文）

`build_context_files_prompt()` 使用 priority 系統，**只有一個會被載入**（first match wins）：

| Priority | 檔案 | 搜尋範圍 | 備註 |
|----------|------|---------|------|
| 1 | `.hermes.md`, `HERMES.md` | CWD 往上到 git root | Hermes-native 專案配置 |
| 2 | `AGENTS.md` | CWD only | 常見 agent instruction 檔 |
| 3 | `CLAUDE.md` | CWD only | Claude Code 相容性 |
| 4 | `.cursorrules`, `.cursor/rules/*.mdc` | CWD only | Cursor 相容性 |

所有 context files 都會經過：
- **Security scan**：檢查 prompt injection 模式
- **Truncation**：上限 20,000 字元（head 70% / tail 20% ratio）
- **YAML frontmatter strip**：`.hermes.md` 的 frontmatter 會被移除

### SOUL.md 的特殊邏輯

當 `load_soul_md()` 成功載入 SOUL.md，會呼叫 `build_context_files_prompt(skip_soul=True)` 防止 SOUL.md 出現兩次。

## Context File Priority System

```python
# Priority: first match wins — only ONE project context loaded
project_context = (
    _load_hermes_md(cwd_path)       # 1. .hermes.md / HERMES.md
    or _load_agents_md(cwd_path)    # 2. AGENTS.md
    or _load_claude_md(cwd_path)    # 3. CLAUDE.md
    or _load_cursorrules(cwd_path)  # 4. .cursorrules / .cursor/rules/*.mdc
)
```

## API-Call-Time-Only Layers

這些明確**不是** cached system prompt 的一部分：
- `ephemeral_system_prompt`
- Prefill messages
- Gateway-derived session context overlays
- Mid-session Honcho recall（注入到 current-turn user message）

## Skills Index

Skills 系統貢獻一個精簡的 skills index 到 prompt：

```markdown
## Skills (mandatory)
Before replying, scan the skills below. If one clearly matches
your task, load it with skill_view(name) and follow its instructions.
...
<available_skills>
  software-development:
    - code-review: Structured code review workflow
    - test-driven-development: TDD methodology
  research:
    - arxiv: Search and summarize arXiv papers
</available_skills>
```

## Concrete Example（完整 assembled system prompt）

```markdown
# Layer 1: Agent Identity (from ~/.hermes/SOUL.md)
You are Hermes, an AI assistant created by Nous Research.
You are an expert software engineer and researcher...

# Layer 2: Tool-aware behavior guidance
You have persistent memory across sessions. Save durable facts using
the memory tool: user preferences, environment details, tool quirks...

# Layer 3: Honcho static block (when active)
[Honcho personality/context data]

# Layer 4: Optional system message
[User-configured system message override]

# Layer 5: Frozen MEMORY snapshot
## Persistent Memory
- User prefers Python 3.12, uses pyproject.toml
- Default editor is nvim
- Timezone: US/Pacific

# Layer 6: Frozen USER profile snapshot
## User Profile
- Name: Alice
- GitHub: alice-dev

# Layer 7: Skills index
## Skills (mandatory)
<available_skills>
  ...
</available_skills>

# Layer 8: Context files
# Project Context
The following project context files have been loaded...
## AGENTS.md
This is the atlas project. Use pytest for testing...

# Layer 9: Timestamp + session
Current time: 2026-03-30T14:30:00-07:00
Session: abc123

# Layer 10: Platform hint
You are a CLI AI Agent. Try not to use markdown...
```

## See also

- [Hermes Agent Core Loop](hermes-agent-core-loop.md)
- [Hermes Memory System](hermes-memory-system.md)
- [Hermes Context Compression](hermes-context-compression.md)
- [Hermes Skills System](hermes-skills-system.md)
