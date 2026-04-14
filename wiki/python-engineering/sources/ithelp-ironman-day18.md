---
title: "Day 18：快取策略：lru_cache、Redis 與失效設計"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, caching, lru_cache, redis, cache-invalidation]
sources: [../../raw/ithelp-ironman-30day/day-18.md]
summary: 分层缓存策略：应用内 lru_cache（零依赖）到跨进程 Redis 共享缓存，Cache-Aside 模式、TTL 失效设计、Stampede 防护与 Redis 重试机制。
related: [ithelp-ironman-series]
---

# Day 18：快取策略：lru_cache、Redis 與失效設計

## Summary

本篇从「量测先行」出发，介绍三层缓存策略：应用内 `functools.lru_cache`（零依赖、单进程）、跨进程 Redis 共享缓存（含 TTL 与失效策略）。涵盖 Cache-Aside 模式、Key 命名（版本前缀）、负向快取、Stampede 防护、重试与降级。强调快取不是万灵药，必须先用 Day 17 工具定位瓶颈再决定是否引入缓存。

## Key Findings

- **何时用缓存**：CPU 密集（重计算）→ lru_cache；I/O 密集（外部 API/数据库）→ Redis；先量测再下刀
- **lru_cache**：`@lru_cache(maxsize=1024, typed=True)`，`.cache_info()` 监控命中/未命中，`.cache_clear()` 部署时清空；不适合多进程共享或隐式状态依赖
- **Redis 分层**：跨实例如 Gunicorn workers、多 Pod 必须用 Redis；`redis.asyncio` 用于 FastAPI/anyio 异步场景，避免在协程中调用同步 I/O
- **Cache-Aside 模式**：先查缓存，miss 时回源重建并写回；`setex` 加 TTL 控制失效；版本前缀（`v1:user:`）管理大规模 key 清理
- **负向快取**（Negative Cache）：查不到的 key 也缓存短 TTL（如 60s），防止缓存击穿
- **Stampede 防护**：用 `tenacity` + `retry` 包装 `rebuild_user`，指数退避避免并发回源；降级回呼 `{"status": "degraded"}`
- **序列化**：推荐使用 Day 15 的 orjson，零依赖高性能
- **工程化**：快取对齐 Day 13 JSON 结构化日志（uid、cache_key、hit/miss、latency_ms 字段），方便观测平台聚合
- **与前文关联**：Day 17 量测后行动；为 Day 19 SQLAlchemy Repository 的可选缓存包装铺垫

## See also

- [ithelp-ironman-day17](ithelp-ironman-day17.md) — 效能觀測：cProfile、py-spy、line-profiler
- [ithelp-ironman-day19](ithelp-ironman-day19.md) — 資料層工程化：SQLAlchemy 2.x 與 Repository Pattern
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
