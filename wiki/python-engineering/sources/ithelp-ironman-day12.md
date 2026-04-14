---
title: "Day 12 - 設定與祕密管理：dotenv、pydantic-settings、雲端祕密服務"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 設定管理, 環境變數, pydantic-settings, 祕密管理, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-12.md]
related: [ithelp-ironman-series]
summary: 建立「設定與祕密」的管理藍圖，從 .env 入門、pydantic-settings 型別安全驗證，到 AWS/GCP/Azure 雲端祕密服務的進階安全防線，以及 CI/CD 與本地開發的分工策略。
---

# Day 12 - 設定與祕密管理：dotenv、pydantic-settings、雲端祕密服務

## 摘要

Day 12 建立設定與祕密管理藍圖，解決「API Key 硬編、.env 誤 commit、環境設定不一致」等問題。從簡單的 .env + python-dotenv，到 pydantic-settings 型別安全驗證，再到 AWS/GCP/Azure 雲端祕密服務的進階安全防線，分三層逐步提升工程化程度。

## 關鍵論點

### 三層演進架構

| 層級 | 工具 | 適用場景 |
|------|------|----------|
| 本地開發 | `.env` + python-dotenv | 個人實驗、小型專案 |
| 工程化專案 | `pydantic-settings` | 團隊專案、分環境管理 |
| 生產/雲端 | AWS Secrets Manager / GCP Secret Manager / Azure Key Vault | 團隊、生產環境 |

### .env 基礎用法

```bash
# .env
APP_ENV=dev
APP_DEBUG=true
DATABASE_URL=postgresql://user:***@localhost/db
```

```python
from dotenv import load_dotenv
import os
load_dotenv()
print(os.getenv("APP_ENV"))
```

- 優點：簡單、直觀、快速上手
- 缺點：缺乏型別驗證，.env 常被誤 commit，祕密容易外洩

### pydantic-settings 工程化設定

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class AppSettings(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="APP_")
    env: str = "dev"
    debug: bool = False
    database_url: str

settings = AppSettings()
```

- 型別安全（不會把 `APP_PORT=abc` 認成 int）
- 明確邊界（必填 vs 有預設值）
- 環境變數有問題直接丟 `ValidationError`
- 可分環境設定檔：`.env.dev` / `.env.prod`

### 雲端祕密服務（AWS 範例）

```python
import boto3
import json
client = boto3.client("secretsmanager")
resp = client.get_secret_value(SecretId="my-app/db")
db_config = json.loads(resp["SecretString"])
```

- 集中管理：祕密不再散落 .env
- 權限管控：IAM 控制誰能讀取
- 自動輪換：部分雲服務能自動更新金鑰

### 專案目錄結構

```
my_project/
├─ src/my_project/
│ ├─ settings.py       # pydantic-settings 模型
│ ├─ secrets.py       # 雲端 Secrets Manager 讀取工具
│ └─ adapters/
│   └─ web.py
├─ .env.example       # 範例環境變數（不含祕密值）
├─ .gitignore          # 忽略 .env、.env.*
└─ tests/
  └─ test_settings.py
```

### CI/CD 整合（GitHub Actions）

```yaml
name: ci
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    env:
      APP_ENV: test
      APP_DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt
      - run: pytest -q
```

### 最佳實務清單

- .env 永遠不要 commit，僅提供 .env.example
- 設定用 pydantic-settings 驗證，避免錯誤流入 runtime
- 祕密用雲端服務管理，並設定最小權限（Least Privilege）
- 在 CI/CD 裡用平台的 Secret 機制，不要偷懶塞 .env
- 在 PR 流程中加入秘密掃描（如 detect-secrets 或 Gitleaks）

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列 Day 12，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Day 11 - 測試策略藍圖](../sources/ithelp-ironman-day11.md) — pytest 測試基礎
- [Day 13 - 結構化日誌](../sources/ithelp-ironman-day13.md) — logging 與 structlog
- [Day 10 - 型別與資料契約](../sources/ithelp-ironman-day10.md) — Pydantic 型別安全
