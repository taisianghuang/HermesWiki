# Log

> 操作日誌，按時間順序記錄所有 wiki 操作。
> 前綴格式：`## [YYYY-MM-DD] <type> | <title>`
> 可用 unix 工具解析：`grep "^## \[" log.md | tail -5`

---

## [2026-04-15] lint | Wiki 健康檢查

- **發現**：共 4 個問題（2 個orphan/wild related、1 個index drift、1 個空 sources）
- **已修復**：
  - ai-security sources/2026-04-15-mythosreadyv9.md：`related: []` → `[Mythos AI 漏洞風暴]`
  - ai-security concepts/2026-04-mythos-ai-vulnerability-storm.md：`related: []` → `[Mythosreadyv9 CSA 報告來源]`
- **需關注**：
  - hermes-agent sources/2026-04-14-hermes-agent-repo.md：空 sources（無 raw 檔案路徑）
  - llm-wiki sources/2026-04-13-llm-wiki-idea.md：空 sources（概念文無 raw 對應）
  - hermes-agent/_index.md 列出 11 sources 比實際 11 sources 多 1 個
- **下次 lint**：2026-05-15

## [2026-04-15] query | Mythos-ready Security Program 所需條件

- **回答來源**：ai-security/sources/2026-04-15-mythosreadyv9.md, ai-security/concepts/2026-04-mythos-ai-vulnerability-storm.md
- **是否存入 analyses/**：是 → 存入 `wiki/ai-security/analyses/2026-04-15-mythosreadyv9-what-is-needed.md`

## [2026-04-15] ingest | CSA Mythosreadyv9 — AI 漏洞風暴戰略簡報

- **主題**：ai-security（新建立）
- **處理的頁面**：
  - Sources: `wiki/ai-security/sources/2026-04-15-mythosreadyv9.md`
  - Concepts: `wiki/ai-security/concepts/2026-04-mythos-ai-vulnerability-storm.md`
  - Topic index: `wiki/ai-security/_index.md`
- **關鍵發現**：
  - Anthropic Claude Mythos Preview 自助發現上千零日漏洞，181 個可用漏洞（72% 成功率）
  - AI 漏洞發現加速軌跡：2025-06 XBOW → 2026-04 Mythos Preview + Project Glasswing
  - Zero Day Clock：時間到利用已壓縮至 < 1 天
  - CSA 提出 13 項風險（4 項 CRITICAL）+ 11 項優先行動（90天計劃）
  - 核心框架：OWASP LLM Top 10 2025、OWASP Agentic Top 10 2026、MITRE ATLAS、NIST CSF 2.0
- **相關現有頁面**：新 topic，無現有頁面
- **創立的 topic**：`wiki/ai-security/`（含 concepts/sources/analyses 三資料夾）

## [2026-04-14] migrate | 結構遷移：扁平 → 階層式主題架構

- **變更類型**：結構遷移（不影響 wiki 內容）
- **變更前**：扁平結構 `wiki/concepts/`、`wiki/sources/`、`wiki/entities/`、`wiki/analyses/`
- **變更後**：階層式主題結構 `wiki/<topic>/{concepts,sources,_index}/`
- **新增資料夾**：`wiki/hermes-agent/`、`wiki/python-engineering/`、`wiki/llm-wiki/`、`output/`
- **刪除資料夾**：`wiki/entities/`、`wiki/analyses/`、`wiki/concepts/`、`wiki/sources/`（皆已清空）
- **更新檔案**：
  - `CLAUDE.md` — schema 更新為階層式架構
  - `index.md` — 改為導向 `wiki/_master-index.md`
  - `wiki/_master-index.md` — 新建頂層總索引
  - `wiki/hermes-agent/_index.md` — 新建主題索引（14 concepts + 11 sources）
  - `wiki/python-engineering/_index.md` — 新建主題索引（2 concepts + 30 sources）
  - `wiki/llm-wiki/_index.md` — 新建主題索引（1 concept + 1 source）
- **更新的 Skills**：`wiki-ingest`、`wiki-lint`、`wiki-query`（皆升至 v2.0.0）
- **主因**：參考他人架構，Topic-based 階層更符合自然分類直覺，擴展性更好

---

## [2026-04-14] ingest | Hermes Agent — ACP Internals、Plugin System、Environments（最後三章）

- **處理的頁面（按 Reading Order 最後三章）**：
  1. ACP Internals → `sources/acp-internals.md` + `concepts/hermes-acp-integration.md` ✅
  2. Plugin System → `sources/plugin-system.md` + `concepts/hermes-plugin-system.md` ✅
  3. Environments / Benchmarks / Data Generation → `sources/environments-rl.md` + `concepts/hermes-environments.md` ✅

- **Raw 檔案新增加**：
  - `raw/hermes-agent/docs/acp-setup.md`（已有）
  - `raw/hermes-agent/environments/README.md`（已有）

- **新建立的 Sources 頁面**：
  - `sources/2026-04-14-hermes-agent-acp-internals.md`
  - `sources/2026-04-14-hermes-agent-plugin-system.md`
  - `sources/2026-04-14-hermes-environments-rl.md`

- **新建立/更新的 Concepts 頁面**：
  - `hermes-acp-integration.md`（新建，ACP 三大編輯器整合）
  - `hermes-plugin-system.md`（新建，四件套架構 + 8 種 hooks）
  - `hermes-environments.md`（新建，三層繼承 + Phase 1/2 + ToolContext）

- **關鍵發現**：
  - ACP 是 Hermes 對編碼助理場景的答案，支援 VS Code/Zed/JetBrains 原生整合
  - Plugin System 讓使用者可新增工具/hooks/技能，與內建工具完全平等
  - pre_llm_call 是唯一可 injection 的 hook，注入到 user message 而非 system prompt
  - Hermes × Atropos 將 Agent 能力延伸到 RL 訓練領域
  - Phase 1（Evaluation/SFT）vs Phase 2（Full RL）的 client-side parser 差異

- **Hermes Agent 全系列完成**：所有 Recommended Reading Order 章節已處理完畢
  - Architecture ✅ → Agent Loop ✅ → Prompt Assembly ✅ → Provider Runtime ✅ → Tools ✅ → Session Storage ✅ → Gateway ✅ → Context Compression ✅ → ACP Internals ✅ → Environments ✅

- **矛盾標註**：無

---

## [2026-04-13] ingest | LLM Wiki — 第二個大腦

- **觸發**: 用戶初始化 LLM Wiki，將 idea file 作為第一個來源攝入
- **處理的頁面**:
  - `wiki/sources/2026-04-13-llm-wiki-idea.md`（新建，來源摘要）
  - `index.md`（更新，新增 Sources 和 Concepts 條目）
  - `log.md`（更新，記錄 ingest）
- **關鍵發現**:
  - 三層架構：Raw Sources / Wiki / Schema
  - 核心洞察：LLM 填補了 Vannevar Bush Memex 願景中「誰來維護」的缺口
  - 與傳統 RAG 的關鍵差異：知識的持續複利 vs 每次從零發現
- **相關概念**: 知識管理、Personal Knowledge Management、Second Brain
- **是否存入 analyses**: 否（這是 source，不是分析）

## [2026-04-14] ingest | Hermes Agent Repo — NousResearch

- **處理的頁面**：
  - Sources: 2026-04-14-hermes-agent-repo.md
  - Concepts: hermes-agent-core-loop.md, hermes-tool-registry.md, hermes-gateway.md, hermes-cli-architecture.md, hermes-toolsets.md, hermes-skills-system.md, hermes-memory-system.md
- **關鍵發現**：
  - Hermes 是自建框架，不使用 LangChain
  - 工具系統採用 Registry 模式，module-level 自動註冊
  - 核心是同步 tool-calling 迴圈（非 async）
  - 支援多平台 Gateway（Telegram、Discord、Slack 等）
  - Skills 系統是自我改進的核心，從經驗中生成技能文件
  - Memory 分短期（Session）和長期（USER.md/MEMORY.md）
- **相關既有頁面**：無直接相關
- **矛盾標註**：無

## [2026-04-14] ingest | Hermes Agent Architecture — Official Docs

- **處理的頁面**：
  - `raw/hermes-agent/docs/architecture.md`（新建原始檔）
  - `wiki/sources/2026-04-14-hermes-agent-architecture-official.md`（新建來源摘要）
- **關鍵發現（與既有 AGENTS.md 的差異）**：
  - 工具數：40+ → 47 tools, 20 toolsets
  - 平台適配器：6 個 → 15 個（新增 matrix, mattermost, email, sms, dingtalk, feishu, wecom, weixin, bluebubbles, webhook）
  - 新增完整行數資訊（run_agent.py ~9,200 行、cli.py ~8,500 行等）
  - 新增 agent/ 目錄模組：context_engine.py、memory_manager.py、memory_provider.py、anthropic_adapter.py
  - 新增 hermes_cli/ 模組：runtime_provider.py、plugins.py、delivery.py、pairing.py、hooks.py、mirror.py、status.py
  - 新增 Plugin System 完整章節
  - 新增 ACP Integration 獨立章節
  - 新增三種完整 Data Flow（CLI Session、Gateway Message、Cron Job）
  - 新增六項 Design Principles
- **相關既有頁面**：hermes-agent-repo.md（作為 source 的 source）
- **矛盾標註**：無
- **修補**：sources 路徑錯誤（`../raw/` → `../../raw/`）

## [2026-04-14] lint

- **Issues found**: 2 total
  - 1 broken related reference (`hermes-agent-tool-registry` → `hermes-tool-registry` in plugin-system source)
  - 58 pages missing `topic:` field after migration to hierarchical structure
- **Fixed immediately**:
  - Fixed broken related ref in `wiki/hermes-agent/sources/2026-04-14-hermes-agent-plugin-system.md`
  - Added `topic: hermes-agent` to all 25 hermes-agent pages
  - Added `topic: python-engineering` to all 32 python-engineering pages
  - Added `topic: llm-wiki` to all 1 llm-wiki pages
- **Needs attention**: None
- **Next lint scheduled**: +30 days

## [2026-04-14] lint | Wiki 健康檢查（完整攝入後）

- **Issues found**: 5 個
  - [已修復] Day 16-20 的 `related:` 欄位缺失（5 個檔案）
- **Fixed immediately**: Day 16-20 各補上 `related: [ithelp-ironman-series]`
- **Needs attention**: 無
- **Next lint scheduled**: 2026-05-14（+30 days）

## [2026-04-14] ingest | Hermes Agent — Recommended Reading Order 完整攝入

- **處理的頁面（按 Reading Order 順序）**：
  1. Architecture Overview → `sources/2026-04-14-hermes-agent-architecture-official.md` ✅
  2. Agent Loop Internals → `raw/docs/agent-loop.md` + `sources/agent-loop.md` + `concepts/hermes-agent-core-loop.md` ✅
  3. Prompt Assembly → `raw/docs/prompt-assembly.md` + `sources/prompt-assembly.md` + `concepts/hermes-prompt-assembly.md` ✅
  4. ~~Provider Runtime Resolution~~（舊進度中已有）
  5. Tools Runtime → `concepts/hermes-tool-registry.md` ✅（舊）
  6. Session Storage → `raw/docs/session-storage.md` + `sources/session-storage.md` + `concepts/hermes-session-storage.md` ✅
  7. Gateway Internals → `raw/docs/gateway-internals.md` + `sources/gateway-internals.md` ✅（Concepts 已豐富化）
  8. Context Compression & Prompt Caching → `raw/docs/context-compression-and-caching.md` + `sources/context-compression.md` + `concepts/hermes-context-compression.md` ✅
  9. ACP Internals → 未處理
  10. Environments, Benchmarks & Data Generation → 未處理

- **Raw 檔案新增加**：
  - `raw/hermes-agent/docs/agent-loop.md`
  - `raw/hermes-agent/docs/prompt-assembly.md`
  - `raw/hermes-agent/docs/context-compression-and-caching.md`
  - `raw/hermes-agent/docs/gateway-internals.md`
  - `raw/hermes-agent/docs/session-storage.md`
  - `raw/hermes-agent/docs/provider-runtime.md`（已有）

- **新建立的 Sources 頁面**：
  - `sources/2026-04-14-hermes-agent-agent-loop.md`
  - `sources/2026-04-14-hermes-agent-prompt-assembly.md`
  - `sources/2026-04-14-hermes-agent-context-compression.md`
  - `sources/2026-04-14-hermes-agent-gateway-internals.md`
  - `sources/2026-04-14-hermes-agent-session-storage.md`
  - `sources/2026-04-14-hermes-agent-provider-runtime.md`

- **新建立/更新的 Concepts 頁面**：
  - `hermes-agent-core-loop.md`（重寫，完整 9,200 行 engine 細節）
  - `hermes-prompt-assembly.md`（新建，10 層 prompt 組裝）
  - `hermes-context-compression.md`（新建，雙層壓縮 + Caching）
  - `hermes-session-storage.md`（新建，SQLite Schema + Lineage）
  - `hermes-provider-runtime.md`（新建，18+ providers）
  - `hermes-gateway.md`（豐富化，加入 Two-Level Guard 等官方內容）

- **關鍵發現（補充之前 AGENTS.md 的不足）**：
  - run_agent.py 約 9,200 行，cli.py 約 8,500 行，gateway/run.py 約 7,500 行
  - 14+ platform adapters（多了 bluebubbles、webhook、api_server）
  - 雙層 compression（Gateway 85% pre-agent safety net + Agent 50% in-loop）
  - Anthropic Prompt Caching 的 system_and_3 滾動策略
  - Context Engine 是可插拔設計（LCM plugin 可替換 default lossy compressor）
  - Session Lineage 透過 parent_session_id 形成鏈
  - SQLite WAL mode + application-level retry + BEGIN IMMEDIATE 處理 contention
  - 8 種 callback surfaces（tool_progress、thinking、reasoning、clarify、step、stream_delta、tool_gen、status）

- **矛盾標註**：無

---

## [2026-04-13] ingest | ithelp Day 1

- **觸發**: 用戶提供 ithelp 鐵人賽 30 天系列文章（Day 1-30），開始批次 ingest
- **本篇處理的頁面**:
  - `wiki/sources/ithelp-ironman-day01.md`（新建，Day 1 來源摘要）
  - `wiki/concepts/ithelp-ironman-series.md`（新建，整個系列的概覽）
  - `wiki/concepts/python-engineering.md`（新建，工程化核心概念）
  - `index.md`（更新，新增 2 concepts、1 source）
  - `log.md`（更新，記錄 ingest）
- **Key findings**:
  - Python 從腳本語言到 ML 主流的演進脈絡
  - 工程化四大價值：可維護性、一致性、可測試性、可擴展性
  - 30 天系列完整路線圖（環境→品質→架構→部署→維運）
  - Hatch 作為統一環境管理工具的出現背景
- **相關頁面**: python-engineering、ithelp-ironman-series
- **是否存入 analyses**: 否（本篇是 source 攝入，不是分析）
- **後續**: Day 2-5 已完成（Day 6-30 待 ingest）

---

## [2026-04-13] ingest | ithelp Day 2-5 — 環境管理、pyproject.toml、目錄結構、Hatch 操作

- **觸發**: 用戶要求 ingest Day 2-5
- **本篇處理的頁面**:
  - `wiki/sources/ithelp-ironman-day02.md`（新建，Day 2 來源摘要）
  - `wiki/sources/ithelp-ironman-day03.md`（新建，Day 3 來源摘要）
  - `wiki/sources/ithelp-ironman-day04.md`（新建，Day 4 來源摘要）
  - `wiki/sources/ithelp-ironman-day05.md`（新建，Day 5 來源摘要）
  - `index.md`（更新，新增 4 sources）
  - `log.md`（更新，記錄 ingest）
- **Key findings**:
  - Day 2: venv/conda/poetry/hatch 比較與場景建議
  - Day 3: pyproject.toml 統一建置與工具設定（PEP 518）
  - Day 4: 專案目錄結構演進（src/ 佈局、不同場景結構）
  - Day 5: Hatch 環境管理指令（create/shell/run/remove）
- **相關頁面**: python-engineering、ithelp-ironman-series
- **是否存入 analyses**: 否（本篇是 source 攝入，不是分析）
- **後續**: Day 6-20 已完成（Day 16-20 本次 ingest），Day 21-25 待 ingest，Day 26-30 已完成

---

## [2026-04-13] ingest | ithelp Day 21-25 — 背景作業、發佈版本、容器化、Dev Container、CI/CD

- **觸發**: 用戶要求 ingest Day 21-25
- **本篇處理的頁面**:
  - `wiki/sources/ithelp-ironman-day21.md`（新建，Day 21 來源摘要）
  - `wiki/sources/ithelp-ironman-day22.md`（新建，Day 22 來源摘要）
  - `wiki/sources/ithelp-ironman-day23.md`（新建，Day 23 來源摘要）
  - `wiki/sources/ithelp-ironman-day24.md`（新建，Day 24 來源摘要）
  - `wiki/sources/ithelp-ironman-day25.md`（新建，Day 25 來源摘要）
  - `index.md`（更新，新增 5 sources）
  - `log.md`（更新，記錄 ingest）
- **Key findings**:
  - Day 21: 背景作業三選一（Celery 企業級、Dramatiq 輕量、asyncio 零 broker）
  - Day 22: Hatch 建置發佈、SemVer 版本管理、Keep a Changelog
  - Day 23: 多階段 Dockerfile（builder/runtime）+ 非 root + 鎖檔可重現
  - Day 24: Dev Container + VS Code + Hatch scripts = 一鍵化 DX
  - Day 25: GitHub Actions CI/CD 範本（uv 鎖檔、多版本矩陣、TestPyPI/PyPI 雙軌）
- **相關頁面**: python-engineering、ithelp-ironman-series
- **是否存入 analyses**: 否（本篇是 source 攝入，不是分析）
- **後續**: Day 26-30 待 ingest

## [2026-04-13] ingest | ithelp Day 16-20 — 非同步基礎、效能觀測、快取、資料層、API 層

- **觸發**: 用戶要求 ingest Day 16-20
- **本篇處理的頁面**:
  - `wiki/sources/ithelp-ironman-day16.md`（新建，Day 16 來源摘要 — asyncio/anyio）
  - `wiki/sources/ithelp-ironman-day17.md`（新建，Day 17 來源摘要 — cProfile/py-spy/line-profiler）
  - `wiki/sources/ithelp-ironman-day18.md`（新建，Day 18 來源摘要 — lru_cache/Redis 失效設計）
  - `wiki/sources/ithelp-ironman-day19.md`（新建，Day 19 來源摘要 — SQLAlchemy 2.x + Repository Pattern）
  - `wiki/sources/ithelp-ironman-day20.md`（新建，Day 20 來源摘要 — FastAPI + Ports/Adapters 六邊形架構）
  - `index.md`（更新，新增 5 sources，Sources 總數 7→12）
  - `log.md`（更新，記錄 ingest）
- **Key findings**:
  - Day 16: async/await 单线程协作式多任务，asyncio 内建 vs anyio 跨生态抽象
  - Day 17: cProfile（全貌）/ py-spy（火焰图）/ line-profiler（逐行）三把剖析利器
  - Day 18: 应用内 lru_cache → 跨进程 Redis Cache-Aside，含 TTL 失效、负向快取、Stampede 防护
  - Day 19: SQLAlchemy 2.x Mapped[T] + Repository Pattern（Port/Adapter 分离）+ 重试/观测
  - Day 20: FastAPI 入站 Adapter + DI + Pydantic v2 + structlog + Hexagonal 架构闭环
- **特殊備註**: raw 檔案 `day-19.md`（內容為 Day 20 API 層）與 `day-20.md`（內容為 Day 19 資料層）檔名互換，已在摘要中標注
- **相關頁面**: python-engineering、ithelp-ironman-series
- **是否存入 analyses**: 否（本篇是 source 攝入，不是分析）
- **後續**: Day 21-25 待 ingest

---

## [2026-04-13] ingest | ithelp Day 26-30 — 安全、部署、監控、AI 協作、總結交付

- **觸發**: 用戶要求 ingest Day 26-30 完成 30 天系列全部攝入
- **本篇處理的頁面**:
  - `wiki/sources/ithelp-ironman-day26.md`（新建，Day 26 來源摘要）
  - `wiki/sources/ithelp-ironman-day27.md`（新建，Day 27 來源摘要）
  - `wiki/sources/ithelp-ironman-day28.md`（新建，Day 28 來源摘要）
  - `wiki/sources/ithelp-ironman-day29.md`（新建，Day 29 來源摘要）
  - `wiki/sources/ithelp-ironman-day30.md`（新建，Day 30 來源摘要）
  - `index.md`（更新，新增 5 sources）
  - `log.md`（更新，記錄 ingest）
- **Key findings**:
  - Day 26: pip-audit × Safety 雙弱點掃描、CycloneDX SBOM、授權白名單治理
  - Day 27: Uvicorn vs Gunicorn+UvicornWorker 場景選擇、Cloud Run 與 K8s 部署設定
  - Day 28: OpenTelemetry Traces/Metrics/Logs 整合、OTel Collector、Prometheus/Tempo
  - Day 29: AI review/doc/changelog 腳本範本、pre-commit/CI/容器三段守門
  - Day 30: 10 條生存法則、待補交付物清單（Checklist/FAQ/模板包）
- **相關頁面**: python-engineering、ithelp-ironman-series
- **是否存入 analyses**: 否（本篇是 source 攝入，不是分析）
- **後續**: ithelp 30 天系列已完成全量 ingest（Day 1-30）

---

## [2026-04-13] lint | Wiki 健康檢查（第二批）

- **Issues found**: 6 個
  - [已修復] sources 路徑錯誤：Day 16-25 的 `sources:` 欄位用了 `../raw/` 而非 `../../raw/`（15個檔案）
  - [已知悉] raw 檔案 day-19.md/day-20.md 內容互換（已在 summary 標註，source page 正確）
  - [已修復] `2026-04-13-llm-wiki-idea.md` 的 `related: []` 為空 → 改為 `related: [python-engineering]`
  - [已知悉] 30 天 source 頁面缺乏 cross-reference（系列文 sequential reading，可接受）
  - [已知悉] Day 16-25 與 Day 1-15/26-30 的 sources 路徑格式曾不一致（已修復）
  - [無問題] Index drift：index.md 與實際檔案完全一致
  - [無問題] Frontmatter 完整性：所有頁面都有 created_by
- **Fixed immediately**:
  - 修復 Day 16-25 的 sources 路徑（`../raw/` → `../../raw/`）
  - 修復 llm-wiki-idea 的 `related: []` → `[python-engineering]`
- **Needs attention**:
  - raw 檔案 day-19/day-20 互換是否要回原始來源修正（不影響 wiki 使用）
  - 可考慮增加更多 cross-reference（如 Day 9 Ruff → Day 10 mypy 的關聯）
- **Next lint scheduled**: 2026-05-13（+30 days）
