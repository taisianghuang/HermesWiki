---
title: "Day 27 - 部署選項速覽：Gunicorn/Uvicorn、Cloud Run / K8s"
type: source
topic: python-engineering
created: 2026-04-13
updated: 2026-04-13
created_by: hermes
tags: [Python, 部署, Uvicorn, Gunicorn, Cloud Run, Kubernetes, Docker, 容器]
sources: [../../raw/ithelp-ironman-30day/day-27.md]
related: [ithelp-ironman-series, python-engineering]
summary: 比較 Uvicorn 單兵 vs Gunicorn+UvicornWorker 的使用場景，介紹 Cloud Run 與 Kubernetes 的最小可用設定，以及健康檢查、降級策略與 CI/CD 整合。
---

# Day 27 - 部署選項速覽：Gunicorn/Uvicorn、Cloud Run / K8s

## 摘要

本篇介紹 Python Web 服務的部署選型：Uvicorn 單兵模式適合 Cloud Run這類平台管理的環境，Gunicorn + UvicornWorker 則提供多程序治理能力。並涵蓋 Cloud Run 與 Kubernetes 的最小可用設定、健康檢查設計、降級策略與 CI/CD 整合。

## 關鍵論點

### 啟動模型選擇

| 場景 | 啟動方式 | 併發與擴縮 | 你要操心的事 |
|------|----------|------------|--------------|
| Cloud Run API | uvicorn | 平台自動擴縮，設 --concurrency | 超時、最小實例、成本、冷啟 |
| K8s 小中型流量 | gunicorn + UvicornWorker | HPA + Requests/Limits | Probes、滾動更新、日誌聚合 |
| 重 I/O、少 CPU | 任一皆可 | 拉高單實例併發 | 外部依賴重試與降級 |
| 重 CPU | 不要硬撐 Web | 把 CPU 料丟背景任務 | 熱點剖析、快取 |

### Uvicorn 直接啟動

```bash
uvicorn my_project.adapters.web.app:app \
 --host 0.0.0.0 --port 8000 \
 --proxy-headers --forwarded-allow-ips='*' \
 --timeout-keep-alive 5
```

### Gunicorn + UvicornWorker 啟動

```bash
gunicorn my_project.adapters.web.app:app \
 -k uvicorn.workers.UvicornWorker \
 --bind 0.0.0.0:8000 \
 --workers ${WORKERS:-$(python - <<'PY'
import os, multiprocessing as m
print(max(2, m.cpu_count()*2 + 1))
PY
)} \
 --threads ${THREADS:-1} \
 --timeout 60 --graceful-timeout 30 \
 --keep-alive 5 --access-logfile '-' --error-logfile '-'
```

### Cloud Run 部署範例

```bash
gcloud run deploy awesome-api \
 --source . \
 --region asia-east1 \
 --platform managed \
 --allow-unauthenticated \
 --cpu=1 --memory=512Mi \
 --concurrency=40 \
 --min-instances=1 --max-instances=50 \
 --timeout=30s \
 --set-env-vars "APP_ENV=prod" \
 --set-secrets "DATABASE_URL=projects/xxx/secrets/db-url:latest" \
 --set-env-vars "APP_CMD=uvicorn my_project.adapters.web.app:app --host 0.0.0.0 --port 8080"
```

### Kubernetes Deployment 核心設定

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: awesome-api
spec:
 replicas: 2
 selector:
   matchLabels: { app: awesome-api }
 template:
   metadata:
     labels: { app: awesome-api }
   spec:
     securityContext:
       runAsUser: 10001
       runAsGroup: 10001
       fsGroup: 10001
     containers:
     - name: web
       image: ghcr.io/you/awesome-api:1.0.0
       env:
       - name: APP_CMD
         value: >-
           gunicorn my_project.adapters.web.app:app
           -k uvicorn.workers.UvicornWorker
           --bind 0.0.0.0:8000
           --workers=3 --timeout=60 --graceful-timeout=30 --keep-alive=5
       - name: DATABASE_URL
         valueFrom:
           secretKeyRef:
             name: app-secrets
             key: database_url
       ports:
       - containerPort: 8000
       resources:
         requests:
           cpu: "250m"
           memory: "256Mi"
         limits:
           cpu: "1"
           memory: "512Mi"
       readinessProbe:
         httpGet: { path: /readyz, port: 8000 }
         periodSeconds: 5
         failureThreshold: 6
       livenessProbe:
         httpGet: { path: /healthz, port: 8000 }
         periodSeconds: 10
         failureThreshold: 3
       securityContext:
         readOnlyRootFilesystem: true
```

### 健康檢查端點

```python
from fastapi import APIRouter, Response, status

router = APIRouter()

@router.get("/healthz")
def healthz():
 return {"ok": True}

@router.get("/readyz")
def readyz():
 ok = True  # 可加入 DB 連線池 ping 等檢查
 return Response(status_code=status.HTTP_200_OK if ok else status.HTTP_503_SERVICE_UNAVAILABLE)
```

## 重要參考

- [Uvicorn](https://www.uvicorn.org/)
- [Gunicorn](https://gunicorn.org/)
- [Google Cloud Run](https://cloud.google.com/run)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

## 系列文關聯

本篇是「30 天 Python 專案工坊」系列的 Day 27，此系列為 **ithelp 鐵人賽 Cyber Edge Runners 團隊**作品，作者為 shothead6062。

---

## See also

- [ithelp-ironman-series](../concepts/ithelp-ironman-series.md) — 整個系列的概覽
- [Python 工程化概念](../concepts/python-engineering.md) — 工程化的核心理念
