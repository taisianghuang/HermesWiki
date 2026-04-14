---
title: Day 23 - 容器化最佳實務：多階段 Dockerfile 與非 root 執行
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, engineering, docker, container, security, devops]
sources: [../../raw/ithelp-ironman-30day/day-23.md]
related: [python-engineering, ithelp-ironman-series]
summary: Docker 多階段建置（builder/runtime 分離）搭配非 root 使用者，打造安全又輕量的容器映像
---

# Day 23 - 容器化最佳實務：多階段 Dockerfile 與非 root 執行

## Summary

本文聚焦容器映像瘦身與安全強化，涵蓋多階段 Dockerfile（builder 安裝依賴、runtime 只帶執行體）、非 root 使用者固定 UID/GID、唯讀檔系統、權限最小化，以及 uv.lock 確保依賴可重現。

## Key Findings

### 多階段 Dockerfile 架構

**Stage 1: builder**
- 安裝編譯工具、原生依賴
- 使用 uv 安裝並鎖定依賴
- 目的：把「會變大的」建置依賴留在 builder

**Stage 2: runtime**
- 只拷貝「結果」：已解決的虛擬環境 + 程式碼
- 體積小、啟動快、攻擊面小

### 非 root 落地做法
- 建立專用帳號 `app:app`，固定 UID/GID（10001）
- 容器啟動改用 `USER app:app`
- 執行時加 `--read-only`，需要寫入就掛 `--tmpfs /tmp`
- 若平台支援，移除多餘 capabilities（如 NET_ADMIN, SYS_PTRACE）

### 關鍵 Dockerfile 設計

```dockerfile
# syntax=docker/dockerfile:1.7

# Stage 1: builder
FROM python:3.12-slim AS builder
RUN apt-get update && apt-get install -y build-essential curl ca-certificates
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --python=/usr/local/bin/python
COPY src/ ./src/

# Stage 2: runtime
FROM python:3.12-slim AS runtime
RUN groupadd -g 10001 app && useradd -m -u 10001 -g 10001 -s /usr/sbin/nologin app
COPY --from=builder /app/src/ /app/src/
COPY --from=builder /root/.cache/uv/venv/*/ /opt/venv/
USER app:app
CMD ["uvicorn", "my_project.adapters.web.app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 可重現依賴
- 建置時使用 `uv.lock`，加上 `--frozen`，確保版本完全一致
- 鎖檔應該跟著原始碼一起版控

###  .dockerignore 必備排除
```
.git, __pycache__, *.pyc, .env*, tests/, notebooks/, dist/, build/, .hatch/, .cache/, .vscode/, .idea/
```

### 運行與除錯
```bash
# 建置
docker build -t myapp:dev .

# 執行（唯讀檔系統 + 暫存寫入）
docker run --rm -p 8000:8000 --read-only --tmpfs /tmp:rw,size=64m myapp:dev
```

### 常見錯誤速查

| 症狀 | 解法 |
|------|------|
| 執行期找不到原生動態庫 | 把必要的系統動態庫也安裝在 runtime |
| 非 root 下啟動失敗 | 在 Dockerfile 中 chown 必要目錄 |
| 每次改一行就重裝依賴 | 先 COPY pyproject.toml/uv.lock，完成安裝後才 COPY src/ |
| 體積仍然過大 | rm -rf /var/lib/apt/lists/*、調整 .dockerignore |

### 安全與體積最佳化清單
- 基底映像用 slim 或最小發行版
- 所有 apt-get install 後清理快取與 headers
- 預設非 root，固定 UID/GID，盡量唯讀
- 移除不必要 capabilities

## See also

- [python-engineering](../concepts/python-engineering.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
