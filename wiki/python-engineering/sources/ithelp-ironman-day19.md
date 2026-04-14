---
title: "Day 19：資料層工程化：SQLAlchemy 2.x 與 Repository Pattern"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, sqlalchemy, repository-pattern, data-layer, hexagonal, architecture]
sources: [../../raw/ithelp-ironman-30day/day-19.md]
summary: SQLAlchemy 2.x 建模（Mapped[T]、select()）与 Repository Pattern 实作，数据层与框架解耦、可测试、可观测、可恢复（重试）、可选缓存包装。
related: [ithelp-ironman-series]
---

# Day 19：資料層工程化：SQLAlchemy 2.x 與 Repository Pattern

## Summary

> ⚠️ **文件命名错位**：`day-20.md` 文件内容实为 Day 19（标题：Day 20 - API 層），`day-19.md` 文件内容实为 Day 20（标题：Day 19 - 資料層）。以下摘要以实际内容为准。

本篇实作数据层的完整工程化方案：SQLAlchemy 2.x 以 `Mapped[T]`/`mapped_column()` 建模、Session 与 Unit of Work（UoW）集中管理事务、Repository Pattern 隔离 ORM 细节（Port/Adapter 分离）。Service 层仅依赖 Repository 接口，可测试、可观测（日志 + 重试）、可选 Redis 缓存包装。

## Key Findings

- **SQLAlchemy 2.x 建模**：`Mapped[T]` 型别标注、`mapped_column()` 声明列、`select()` 查询、`session.get()` 按 ID 查找；与 Day 10 Pydantic v2 型别策略兼容
- **Session / UoW**：`sessionmaker` + context manager `unit_of_work()`，自动 commit/rollback；`expire_on_commit=False` 避免脱管；生产环境建议用 Alembic 管理 schema 演进（Day 7 可重再现）
- **Repository Pattern**：
  - Port（介面）：`domain/repositories.py` 定义 `UserRepository` 协议，业务语言
  - Adapter：`infrastructure/db/repositories.py` 实作 `SqlAlchemyUserRepository`，只看到 ORM
  - Service：`services/user_service.py` 仅依赖 Port，与 HTTP/ORM 完全解耦
- **重试包装**：`tenacity` + `stop_after_attempt(3)` + `wait_exponential` 包装 `OperationalError`（瞬断、连接池抖动）
- **Cache-Aside 包装**：`CachedUserRepository`（可选），用 `redis.asyncio` 缓存 `get`，TTL 300s，负向快取 60s
- **观测性**：`@timed` 装饰器统一记录 `fn`、`ms` 字段到 structlog JSON
- **测试**：in-memory SQLite + `sessionmaker` fixture，单元测试不碰真实 DB
- **异步版本**：可选 `create_async_engine` + `async_sessionmaker` + `AsyncSession`
- **工程化**：与 Day 12 pydantic-settings、Day 13 structlog、Day 14 tenacity、Day 15 orjson 完全对齐
- **为 Day 20 做铺垫**：本篇建立 Repository，Day 20 FastAPI 将作为入站 Adapter 接入

## See also

- [ithelp-ironman-day18](ithelp-ironman-day18.md) — 快取策略：lru_cache、Redis 與失效設計
- [ithelp-ironman-day20](ithelp-ironman-day20.md) — API 層：用 FastAPI 落實 Ports/Adapters
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
