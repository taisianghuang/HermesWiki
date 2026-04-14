---
title: "Day 15 - 序列化與設定格式：orjson、YAML、TOML 實務"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 序列化, JSON, orjson, YAML, TOML, 設定格式, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-15.md]
related: [ithelp-ironman-series]
summary: 介紹資料序列化與設定格式的分工：orjson 高效能 JSON 處理適用於 API 與快取、YAML 適合人類維護的 Infra/CI/CD 設定、TOML 是 Python 專案設定的官方標準（PEP 621），並討論何時用哪種格式。
---

# Day 15 - 序列化與設定格式：orjson、YAML、TOML 實務

## 摘要

Day 15 清楚定位三種格式的適用場景：orjson（JSON）用於高效能 API 與快取、YAML 用於人類維護的 Infra/CI/CD 設定、TOML 是 Python 專案設定的官方標準。核心精神是「格式與責任分離」——設定與程式分離、資料與格式分離。

## 關鍵論點

### 三種格式的角色分工

| 場景 | 推薦格式 | 理由 |
|------|----------|------|
| API 資料交換 / 快取 | **orjson (JSON)** | 高效能、跨語言相容 |
| 人類維護的 Infra / CI/CD 設定 | **YAML** | 可讀性高、支援註解 |
| Python 專案本身的設定 | **TOML** | 官方標準、簡單結構 |

### orjson：高效能 JSON

```python
import orjson
from datetime import datetime

data = {"id": 1, "ts": datetime.now()}
serialized = orjson.dumps(data)   # → bytes
parsed = orjson.loads(serialized)  # → dict
```

特點：
- 比標準 `json` 快數倍，適合高流量 API
- 強制 UTF-8，避免亂碼
- 原生支援 datetime / numpy，不用自己轉字串

工程實務：
- API Response → orjson 幾乎是必備
- 快取層（Redis、檔案）→ `orjson.dumps / loads`
- 若只依賴標準庫 → 退而用 `json`，但效能差距顯著

### YAML：人類友善的設定格式

```yaml
app:
  name: demo
  debug: true
  database:
    url: postgresql://user:***@localhost/db
    pool_size: 10
```

```python
import yaml
with open("config.yaml") as f:
    config = yaml.safe_load(f)
```

優點：
- 可讀性高，支援巢狀結構、註解
- 生態成熟，各語言幾乎都有 parser

缺點：
- 縮排錯誤很常見
- 曾有安全漏洞（**千萬別用 `yaml.load`，要用 `safe_load`**）

適用場景：CI/CD、部署、Infra 設定；搭配 Pydantic 做 Schema 驗證

### TOML：Python 專案設定標準

```toml
[app]
name = "demo"
debug = true

[database]
url = "postgresql://user:***@localhost/db"
pool_size = 10
```

```python
import tomllib  # Python 3.11 內建
with open("config.toml", "rb") as f:
    config = tomllib.load(f)
```

特點：
- 單純明確，比 YAML 更直觀，幾乎沒有隱式陷阱
- PEP 官方背書，Python 3.11 內建
- `pyproject.toml` 就是 TOML 格式（PEP 518/621）

### 專案目錄對應

```
my_project/
├─ pyproject.toml      # 專案設定（TOML）
├─ config.yaml         # 部署/Infra 設定（YAML）
├─ cache/
│  └─ response.json   # 儲存 JSON 序列化結果（orjson）
└─ src/my_project/
   ├─ settings.py      # 載入並驗證設定（pydantic-settings）
   └─ serializers.py   # 集中處理 orjson dumps/loads
```

### 最佳實務

- JSON/orjson：效率與通用性，最適合資料交換
- YAML：人類友善，適合維護大量設定
- TOML：專案設定的官方標準
- 格式與責任分離：設定與程式分離、資料與格式分離

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列 Day 15，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Day 12 - 設定與祕密管理](../sources/ithelp-ironman-day12.md) — pydantic-settings 與設定驗證
- [Day 16 - 非同步基礎](../sources/ithelp-ironman-day16.md) — asyncio / anyio 高併發
- [Day 11 - 測試策略藍圖](../sources/ithelp-ironman-day11.md) — 測試與覆盖率
