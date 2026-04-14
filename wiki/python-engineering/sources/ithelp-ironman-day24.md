---
title: Day 24 - 開發體驗：Dev Container / VS Code 與 Hatch scripts
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [python, engineering, devcontainer, vscode, hatch, developer-experience]
sources: [../../raw/ithelp-ironman-30day/day-24.md]
related: [python-engineering, ithelp-ironman-series]
summary: Dev Container 鎖住開發環境、VS Code 整合、Hatch scripts 一鍵化，打造「任何人打開就能寫」的 DX
---

# Day 24 - 開發體驗：Dev Container / VS Code 與 Hatch scripts

## Summary

本文解說如何用 Dev Container 把開發環境容器化、VS Code 整合偵錯與任務、Hatch scripts 把雜事一鍵化。目標：任何人 clone 後按「Reopen in Container」就能開始寫，環境、工具、命令全部一致。

## Key Findings

### Dev Container 核心價值
- 把開發環境（套件版本、系統依賴、工具鏈）寫進定義檔
- 新人 clone 下來，按「Reopen in Container」，環境就緒
- 與「src/ + tests/」目錄骨架、可安裝套件設計是同一脈絡

### 專案結構
```
my_project/
├─ .devcontainer/
│ ├─ devcontainer.json
│ ├─ Dockerfile（可選：若你想沿用多階段 Dockerfile 思路）
│ └─ postCreate.sh（可選）
├─ pyproject.toml
├─ uv.lock
├─ src/my_project/
└─ tests/
```

### devcontainer.json 最小可用範例
```json
{
  "name": "my_project dev",
  "build": { "dockerfile": "Dockerfile", "context": ".." },
  "remoteUser": "app",
  "customizations": {
    "vscode": {
      "extensions": ["ms-python.python", "ms-python.vscode-pylance", "charliermarsh.ruff", "ms-toolsai.jupyter"],
      "settings": { "python.defaultInterpreterPath": "/opt/venv/bin/python", "python.testing.pytestEnabled": true }
    }
  },
  "postCreateCommand": "hatch run dev:bootstrap"
}
```

### Hatch scripts 把雜事變一行
```toml
[project.optional-dependencies]
dev = ["pytest", "pytest-cov", "ruff", "black", "mypy", "httpx", "fastapi"]

[tool.hatch.envs.dev]
features = ["dev"]

[tool.hatch.envs.dev.scripts]
bootstrap = ["ruff --version", "python -c \"print('venv ready')\""]
check = ["ruff check .", "black --check .", "mypy src/", "pytest -q"]
test = "pytest -q"
fmt = "black ."
type = "mypy src/"
serve = "uvicorn my_project.adapters.web.app:app --reload --port 8000"
```

### VS Code 整合

**tasks.json**（把 Hatch 指令變成快捷鍵任務）
```json
{ "label": "check", "type": "shell", "command": "hatch run dev:check", "group": "build" }
```

**launch.json**（API 服務即時偵錯）
```json
{ "name": "API (uvicorn)", "type": "python", "module": "uvicorn",
  "args": ["my_project.adapters.web.app:app", "--reload", "--port", "8000"],
  "python": "/opt/venv/bin/python" }
```

### 一次到位的「打開就能寫」流程
1. Clone 專案，Reopen in Container
2. 自動 postCreate → Hatch 執行 dev:bootstrap
3. `hatch run dev:check` 一鍵執行 ruff、black、mypy、pytest
4. `hatch run dev:serve` 用 uvicorn 啟動服務
5. 開啟 REST Client / 前端對接測通路

### Debug 食譜

| 症狀 | 解法 |
|------|------|
| 換一行就重裝半小時 | 先 COPY pyproject.toml/uv.lock，安裝完再 COPY src/ |
| 容器內權限錯誤 | 事先 chown 目錄或掛載 volume 時指定擁有者 |
| 容器過大或建置太慢 | 清 apt 快取、調整 .dockerignore |
| 本機與 CI 行為不一致 | 使用 uv.lock 並 --frozen 同步依賴 |

### Nox 延伸：多 Python 版本驗證
```python
@nox.session(python=["3.10","3.11","3.12"])
def tests(session):
    session.install(".[dev]")
    session.run("pytest","-q")
```

## See also

- [python-engineering](../concepts/python-engineering.md)
- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md)
