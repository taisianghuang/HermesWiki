---
title: "Day 20：API 層：用 FastAPI 落實 Ports/Adapters（六邊形味道）"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, fastapi, hexagonal-architecture, ports-and-adapters, api, dependency-injection]
sources: [../../raw/ithelp-ironman-30day/day-20.md]
summary: 用 FastAPI 作为入站 Adapter 实作六边形架构，完整整合 Day 10 Pydantic v2、Day 13 structlog、Day 14 重试降级、Day 19 Repository，与 Service/Port 完全解耦。
related: [ithelp-ironman-series]
---

# Day 20：API 層：用 FastAPI 落實 Ports/Adapters（六邊形味道）

## Summary

> ⚠️ **文件命名错位**：`day-19.md` 文件内容实为 Day 20（标题：Day 19 - 資料層），`day-20.md` 文件内容实为 Day 19（标题：Day 20 - API 層）。以下摘要以实际内容为准。

本篇是「入站 Adapter × 应用服务 × 数据层」闭环的最后一环：用 FastAPI 作为入站 Adapter，透过 Dependency Injection 将 Day 19 的 `UserRepository`（Port）接入 API 路由。完整整合 Day 10 Pydantic v2 数据契约、Day 13 structlog JSON 日志、Day 14 tenacity 重试降级、Day 18 Redis 缓存。提供完整目录结构、TestClient 集成测试与 Hatch 一键启动。

## Key Findings

- **六边形架构**：Web（FastAPI）= 入站 Adapter，Repository = 出站 Adapter，Service + Port = 核心业务规则；好处：可替换 Web 框架、ORM、缓存而不动用例
- **目录结构**：`src/my_project/adapters/web/` 下放置 `app.py`、`routers/users.py`、`schemas.py`、`deps.py`、`errors.py`、`middleware.py`；`domain/` 下 Port 介面；`services/` 下应用服务；`infrastructure/db/` 下 ORM Adapter
- **Pydantic v2 契约**：`CreateUserIn`（入站验证）、`UserOut`（出站模型）、`UsersPage`（分页）；`Annotated` + `Field` 精细验证，与 Day 10 一致
- **Dependency Injection**：`deps.py` 定义 `get_session()` → `get_user_repo()` → `get_user_service()` 链，FastAPI `Depends()` 自动注入
- **Lifespan**：`@asynccontextmanager` 管理启动/关闭事件（如连接池 warm-up、metrics 启动）
- **路由**：仅与 Service 说话，不碰 ORM/HTTP 细节；`register_new_user`、`get_user`、`list_users` 用例语义清晰
- **异常映射**：`ConflictError` → 409、`NotFoundError` → 404、`Exception` → 500 + 降级响应；结构化日志记录
- **Middleware**：请求追踪（`x-request-id`）、计时（`ms`）、JSON 结构化日志，与 Day 13 一致
- **可选增强**：Day 18 Cache-Aside 包装 Repository、Day 14 tenacity 重试 `OperationalError`、Day 15 orjson 替换默认序列化
- **测试**：`TestClient(app)` 端到端集成测试，`monkeypatch` 隔离 fixture
- **Hatch scripts**：`serve = "uvicorn my_project.adapters.web.app:app --reload --port 8000"`，`check` 跑 ruff + mypy，`test` 跑 pytest

## See also

- [ithelp-ironman-day19](ithelp-ironman-day19.md) — 資料層工程化：SQLAlchemy 2.x 與 Repository Pattern
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
