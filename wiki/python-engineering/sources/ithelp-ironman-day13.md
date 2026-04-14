---
title: "Day 13 - 結構化日誌：logging/structlog 與 JSON Log"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 日誌, logging, structlog, JSON, 雲端監控, 鐵人賽]
sources: [../../raw/ithelp-ironman-30day/day-13.md]
related: [ithelp-ironman-series]
summary: 從傳統字串日誌進化到結構化日誌，介紹 Python 內建 logging 模組、structlog 的 key-value 結構化日誌，以及 JSON Log 與雲端監控平台（ELK/Loki/Datadog）的整合實踐。
---

# Day 13 - 結構化日誌：logging/structlog 與 JSON Log

## 摘要

Day 13 建立結構化日誌藍圖，讓日誌從「文字堆疊」進化成「可被機器分析的資料流」。涵蓋 Python 內建 logging 模組、structlog 的 key-value 結構化日誌、JSON Log 格式，以及與 ELK / Grafana Loki / Datadog 等雲端監控平台的整合。

## 關鍵論點

### 傳統 logging 模組

```python
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)
logger.info("user logged in: %s", "alice")
logger.error("failed to fetch data: %s", "timeout")
```

輸出：
```
INFO:__main__:user logged in: alice
ERROR:__main__:failed to fetch data: timeout
```

- 優點：標準庫內建，零依賴，支援 log level、formatter、handler
- 缺點：單純字串，無法被 JSON parser 或監控平台直接分析

### structlog：結構化 key-value 日誌

```python
import structlog
logger = structlog.get_logger()
logger.info("user_logged_in", user="alice", ip="127.0.0.1")
logger.error("db_error", query="SELECT * FROM users", error="timeout")
```

JSON 格式化後輸出：
```json
{"event": "user_logged_in", "user": "alice", "ip": "127.0.0.1", "level": "info"}
{"event": "db_error", "query": "SELECT * FROM users", "error": "timeout", "level": "error"}
```

- Key-value 結構，方便過濾與分析
- 能與 logging 整合，不用重寫所有程式碼
- 適合導入 ELK、Grafana Loki 等監控平台

### JSON Log 與雲端部署

容器化與雲端部署（Kubernetes）最佳實務：

```python
import logging
import structlog

logging.basicConfig(level=logging.INFO)
structlog.configure(
    processors=[
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer()
    ]
)
logger = structlog.get_logger()
logger.info("payment_success", order_id="A123", amount=100)
```

輸出：
```json
{"event": "payment_success", "order_id": "A123", "amount": 100, "level": "info", "timestamp": "2025-09-21T16:00:00Z"}
```

- stdout：由容器 runtime 收集，避免寫檔案
- JSON 格式：方便 Fluent Bit、Vector、Datadog agent 解析
- 天然支援 ELK / Loki / Datadog / CloudWatch

### 專案整合：logging_config.py

```
my_project/
├─ src/my_project/
│ ├─ __init__.py
│ ├─ logging_config.py    # 集中設定
│ ├─ core.py
│ └─ adapters/
│   └─ web.py
└─ tests/
  └─ test_logging.py
```

### 日誌測試（caplog fixture）

```python
def test_logging_json(caplog):
    import json
    from my_project.logging_config import setup_logging
    import structlog

    setup_logging()
    logger = structlog.get_logger()
    logger.info("test_event", key="value")

    log = json.loads(caplog.text.splitlines()[0])
    assert log["event"] == "test_event"
    assert log["key"] == "value"
```

### 最佳實務清單

- `logging`：小型專案或快速原型
- `structlog`：結構化、可擴充，適合進階專案
- JSON log + stdout：容器化與雲端最佳實務
- 集中設定檔：避免每個模組自行定義格式
- 測試日誌：確保關鍵事件被記錄

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列 Day 13，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Day 11 - 測試策略藍圖](../sources/ithelp-ironman-day11.md) — 測試基礎與 caplog 驗證
- [Day 14 - 例外分層、重試與降級](../sources/ithelp-ironman-day14.md) — 錯誤處理與 structlog 整合
- [Day 12 - 設定與祕密管理](../sources/ithelp-ironman-day12.md) — 雲端祕密服務
