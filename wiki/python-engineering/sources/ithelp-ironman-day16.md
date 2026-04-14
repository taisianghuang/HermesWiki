---
title: "Day 16：非同步基礎：asyncio / anyio 的落地心法"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, async, asyncio, anyio, concurrency]
sources: [../../raw/ithelp-ironman-30day/day-16.md]
summary: asyncio 與 anyio 的非同步 I/O 核心概念、async/await 語法、以及與同步阻塞混合時的工程注意事項。
related: [ithelp-ironman-series]
---

# Day 16：非同步基礎：asyncio / anyio 的落地心法

## Summary

本篇介紹 Python 非同步 I/O 的核心概念，对比 asyncio（官方内建）与 anyio（跨生态抽象层）的用法。强调 async/await 是单线程协作式多任务，目标是在 I/O 等待期间让 CPU 切换到其他任务，而非并行执行。包含工程化注意事项：禁止在 async 函数中混用同步阻塞 I/O、pytest-asyncio 测试写法、以及透过 Hatch 一键化 async 测试环境。

## Key Findings

- **async/await 本质**：单线程协作式多任务，不等于多进程（multiprocessing）或多线程（threading）；核心价值是避免 CPU 在 I/O 等待时空转
- **asyncio**：Python 3.5+ 内建，`async def` 声明协程，`await` 暂停不阻塞 event loop，`asyncio.gather()` 并发执行多个协程
- **anyio**：抽象层，同时支持 asyncio / trio，提供 `TaskGroup` 管理并行任务，自动错误传播，FastAPI/Starlette/Trio 通用
- **工程禁忌**：在 async 函数里调用同步 `requests.get()` 会卡死整个 event loop，必须换用异步库（httpx、aiosqlite）或 `anyio.to_thread.run_sync`
- **测试**：`pytest.mark.asyncio` + `pytest-asyncio` 插件支持直接写异步测试
- **Hatch 集成**：可在 `pyproject.toml` 建立独立 `[tool.hatch.envs.async]` 环境，脚本 `test = "pytest -q tests/async"`
- **目录建议**：将异步 I/O 集中在 `src/my_project/async_ops.py`，适配器放 `adapters/web.py`（FastAPI/httpx）
- **与前文关联**：承接 Day 15 结构化序列化，为 Day 17 效能观测定调

## See also

- [ithelp-ironman-day17](ithelp-ironman-day17.md) — 效能觀測：cProfile、py-spy、line-profiler
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
