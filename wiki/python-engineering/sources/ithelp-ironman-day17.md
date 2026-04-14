---
title: "Day 17：效能觀測：cProfile、py-spy、line-profiler"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, profiling, performance, cprofile, py-spy, line-profiler]
sources: [../../raw/ithelp-ironman-30day/day-17.md]
summary: 使用 cProfile、py-spy、line-profiler 三把利器进行性能剖析，区分 CPU 密集与 I/O 密集问题，以及与 Hatch/Nox 的一键化集成。
related: [ithelp-ironman-series]
---

# Day 17：效能觀測：cProfile、py-spy、line-profiler

## Summary

本篇建立「先量测再优化」的工程化思维，介绍 Python 世界三把性能剖析利器：cProfile（标准库全局鸟瞰）、py-spy（低干扰采样式，可生成火焰图）、line-profiler（逐行热点分析）。区分 CPU 密集与 I/O 密集问题的适用工具，并展示如何透过 pyproject.toml 的 optional-dependencies 和 Hatch scripts 将三者一键化集成到开发工作流。

## Key Findings

- **cProfile**：标准库自带，`python -m cProfile -o profile.out your_app.py`；`pstats` 交互式分析，`sort_stats("tottime")` 查纯 Python 时间，`sort_stats("cumtime")` 查含调用链累积时间；适合 CI 基准测试留存
- **py-spy**：无需改代码即可挂到运行中进程，`py-spy top --pid <PID>` 实时观察；`py-spy record -o profile.svg --duration 30` 生成火焰图；采样式低干扰，适合线上 debug
- **line-profiler**：在目标函数加 `@profile` 装饰器，`kernprof -l -v script.py` 输出逐行执行次数与耗时；适合已知瓶颈函数后进一步锁定「哪一行」
- **工具选用逻辑**：CPU 烧在哪 → cProfile/py-spy；哪一行最慢 → line-profiler；I/O 拖慢 → 回检异步设计（Day 16）
- **Hatch × Nox 集成**：`[project.optional-dependencies]` 加 `perf` 组（含 py-spy、line-profiler、snakeviz），`[tool.hatch.envs.perf]` 环境脚本 `cprof`/`pyspy`/`lprof`，noxfile.py 对应三个 session
- **与前文关联**：承接 Day 16 异步观喘，为 Day 18 快取策略提供「量测先行」的方法论基础

## See also

- [ithelp-ironman-day16](ithelp-ironman-day16.md) — 非同步基礎：asyncio / anyio
- [ithelp-ironman-day18](ithelp-ironman-day18.md) — 快取策略：lru_cache、Redis 與失效設計
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
