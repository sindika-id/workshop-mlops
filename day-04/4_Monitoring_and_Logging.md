# 4. Monitoring and Logging

## 🎯 Learning Objectives
- Learn why monitoring is essential for deployed ML models.  
- Implement structured logging in FastAPI.  
- Use Prometheus + Grafana for monitoring ML services.  

---

## 📘 Why Monitor ML Models?

- **Data Drift**: Input data may change over time → predictions degrade.  
- **Model Decay**: Performance decreases as environment changes.  
- **Service Health**: Track API uptime, latency, errors.  

Monitoring ensures **reliability** and **trust** in ML systems.  

---

## 🛠 Step 1: Structured Logging in FastAPI

Add logging to `app/main.py`:

```python
import logging
from fastapi import FastAPI

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s"
)
logger = logging.getLogger(__name__)

app = FastAPI()

@app.get("/health")
def health_check():
    logger.info("Health check called")
    return {"status": "ok"}
```

![alt text](images/4_Monitoring_and_Logging/1_add_logging.png)

Logs will include timestamps, log levels, and messages.  

---

## 🛠 Step 2: Request Logging Middleware

```python
from fastapi import Request

@app.middleware("http")
async def log_requests(request: Request, call_next):
    logger.info(f"Incoming request: {request.method} {request.url}")
    response = await call_next(request)
    logger.info(f"Completed with status {response.status_code}")
    return response
```

![alt text](images/4_Monitoring_and_Logging/2_request_logging_middleware.png)

---

## 🛠 Step 3: Expose Metrics with Prometheus

Install:
```bash
pip install prometheus-fastapi-instrumentator
```
![alt text](images/4_Monitoring_and_Logging/3_install_prometheus.png)

Add to FastAPI app:

```python
from prometheus_fastapi_instrumentator import Instrumentator

Instrumentator().instrument(app).expose(app)
```

![alt text](images/4_Monitoring_and_Logging/3_prometheus_main.png)

Now metrics available at 👉 `/metrics` (Prometheus format).  

---

## 🛠 Step 4: Docker Compose with Prometheus + Grafana

Add services to `docker-compose.yml`:

```yaml
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
```

![alt text](images/4_Monitoring_and_Logging/4_add_service_to_docker_compose.png)

`prometheus.yml` config:
```yaml
scrape_configs:
  - job_name: "fastapi"
    static_configs:
      - targets: ["api:8000"]
```

![alt text](images/4_Monitoring_and_Logging/4_prometheus_yml_config.png)

---

## 🛠 Step 5: Visualize in Grafana

- Open 👉 http://localhost:3000

  Login with username and password admin:

![alt text](images/4_Monitoring_and_Logging/5_login_grafana.png)

- Add Prometheus as data source (`http://prometheus:9090`).

![alt text](images/4_Monitoring_and_Logging/5_find_data_sources.png)

  Add data source, then add Prometheus:

![alt text](images/4_Monitoring_and_Logging/5_add_data_source.png)

  Add (`http://prometheus:9090`) as connection, then scroll to bottom and save it:

![alt text](images/4_Monitoring_and_Logging/5_prometheus_connection.png)

- Import dashboard for FastAPI metrics.

  Select dashboard in sidebar:

![alt text](images/4_Monitoring_and_Logging/5_select_dashboard.png)

  Click import dashboard:
  
![alt text](images/4_Monitoring_and_Logging/5_import_dashboard.png)

  Load dashboard id 22676:

![alt text](images/4_Monitoring_and_Logging/5_load_dashboard_22676.png)

  Import the dashboard:

![alt text](images/4_Monitoring_and_Logging/5_import_the_dashboard.png)

![alt text](images/4_Monitoring_and_Logging/5_grafana_dashboard.png)

---

## ✅ Summary
- Logging ensures visibility into API calls and system behavior.  
- Prometheus scrapes metrics from FastAPI `/metrics`.  
- Grafana visualizes latency, error rates, and custom ML metrics.  
- Monitoring keeps ML services reliable in production.  