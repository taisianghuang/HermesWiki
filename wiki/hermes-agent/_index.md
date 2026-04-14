# hermes-agent

> Hermes Agent 框架的完整知識庫，含架構設計、開發指南、工具系統。

## Concepts（概念頁）

| 頁面 | 摘要 | 更新 |
|------|------|------|
| [hermes-agent-core-loop](concepts/hermes-agent-core-loop.md) | Hermes 核心同步 tool-calling 迴圈，~9,200 行，含 API Modes、Turn Lifecycle、Callbacks、Budget、Fallback | 2026-04-14 |
| [hermes-tool-registry](concepts/hermes-tool-registry.md) | Hermes 工具系統的 Registry 模式，每個工具檔案在 import 時自動註冊 | 2026-04-14 |
| [hermes-gateway](concepts/hermes-gateway.md) | Hermes 多平台訊息適配器系統，支援 14+ 平台含 Telegram、Discord、Slack | 2026-04-14 |
| [hermes-cli-architecture](concepts/hermes-cli-architecture.md) | Hermes CLI 使用 Rich + prompt_toolkit 建構，透過 CommandDef 集中定義斜線指令 | 2026-04-14 |
| [hermes-toolsets](concepts/hermes-toolsets.md) | Hermes 工具分組系統，支援工具集繼承和動態啟用/停用 | 2026-04-14 |
| [hermes-skills-system](concepts/hermes-skills-system.md) | Hermes 自我改進核心，Agent 從經驗中創建和改進技能（Markdown 文件） | 2026-04-14 |
| [hermes-memory-system](concepts/hermes-memory-system.md) | Hermes 多層次記憶系統，包含短期 Session 和長期 Persistent Memory | 2026-04-14 |
| [hermes-prompt-assembly](concepts/hermes-prompt-assembly.md) | Hermes 10 層 cached system prompt 組裝：SOUL.md、Memory snapshot、Context files、Skills index | 2026-04-14 |
| [hermes-context-compression](concepts/hermes-context-compression.md) | 雙層壓縮（Gateway 85% + Agent 50%）+ Anthropic Prompt Caching（system_and_3）| 2026-04-14 |
| [hermes-session-storage](concepts/hermes-session-storage.md) | SQLite WAL mode 持久化（sessions、messages、FTS5）、Session Lineage、Write Contention Handling | 2026-04-14 |
| [hermes-provider-runtime](concepts/hermes-provider-runtime.md) | 跨 CLI/Gateway/Cron 的 Provider Resolution，18+ providers，Resolution Precedence | 2026-04-14 |
| [hermes-plugin-system](concepts/hermes-plugin-system.md) | Hermes Plugin 四件套架構（manifest/schemas/handlers/registration）、8種 Hooks、Skill Bundling、Env Gates | 2026-04-14 |
| [hermes-acp-integration](concepts/hermes-acp-integration.md) | Hermes ACP（Agent Client Protocol）讓 Agent 运行在 VS Code、Zed、JetBrains 編輯器中 | 2026-04-14 |
| [hermes-environments](concepts/hermes-environments.md) | Hermes × Atropos RL 訓練整合，三層繼承、ToolContext、Phase 1/2 雙階段、12 種 parsers | 2026-04-14 |

## Sources（來源摘要）

| 頁面 | 摘要 | 更新 |
|------|------|------|
| [2026-04-14-hermes-agent-repo](sources/2026-04-14-hermes-agent-repo.md) | NousResearch 的 Hermes Agent 開源 repo，支援多平台 Gateway、40+工具、內建學習循環 | 2026-04-14 |
| [2026-04-14-hermes-agent-architecture-official](sources/2026-04-14-hermes-agent-architecture-official.md) | 官方 Architecture 文檔，含 System Overview、Data Flow、Design Principles | 2026-04-14 |
| [2026-04-14-hermes-agent-agent-loop](sources/2026-04-14-hermes-agent-agent-loop.md) | 官方 Agent Loop 文檔，~9,200 行核心 engine，含 API Modes、Turn Lifecycle | 2026-04-14 |
| [2026-04-14-hermes-agent-prompt-assembly](sources/2026-04-14-hermes-agent-prompt-assembly.md) | 官方 Prompt Assembly 文檔，10 層 cached system prompt、SOUL.md | 2026-04-14 |
| [2026-04-14-hermes-agent-context-compression](sources/2026-04-14-hermes-agent-context-compression.md) | 官方 Context Compression 文檔，雙層壓縮（85%/50%）、4 階段 algorithm | 2026-04-14 |
| [2026-04-14-hermes-agent-gateway-internals](sources/2026-04-14-hermes-agent-gateway-internals.md) | 官方 Gateway Internals 文檔，14+ Platform Adapters、Two-Level Guard | 2026-04-14 |
| [2026-04-14-hermes-agent-session-storage](sources/2026-04-14-hermes-agent-session-storage.md) | 官方 Session Storage 文檔，SQLite Schema、Schema Migrations、Write Contention | 2026-04-14 |
| [2026-04-14-hermes-agent-provider-runtime](sources/2026-04-14-hermes-agent-provider-runtime.md) | 官方 Provider Runtime 文檔，18+ providers、Resolution Precedence | 2026-04-14 |
| [2026-04-14-hermes-agent-acp-internals](sources/2026-04-14-hermes-agent-acp-internals.md) | Hermes ACP 編輯器整合指南，含 VS Code/Zed/JetBrains 設定 | 2026-04-14 |
| [2026-04-14-hermes-agent-plugin-system](sources/2026-04-14-hermes-agent-plugin-system.md) | Hermes Plugin 開發完整指南，4件套架構、8種 hooks | 2026-04-14 |
| [2026-04-14-hermes-environments-rl](sources/2026-04-14-hermes-environments-rl.md) | Hermes × Atropos RL 整合文檔，三層繼承、ToolContext、Phase 1/2 | 2026-04-14 |

## Statistics
- **Concepts**: 14
- **Sources**: 11
- **Total**: 25
- **Last updated**: 2026-04-14
