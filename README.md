# OpenTelemetry Observability Stack

Complete observability setup with OpenTelemetry, Prometheus, Grafana, Jaeger, and Loki.

## Architecture

- **Flask Application**: Instrumented with OpenTelemetry
- **OpenTelemetry Collector**: Receives, processes, and exports telemetry data
- **Prometheus**: Metrics storage and querying
- **Grafana**: Visualization dashboard
- **Jaeger**: Distributed tracing
- **Loki**: Log aggregation
- **Promtail**: Log shipping

## Features

✅ **Traces**: Distributed tracing with Jaeger  
✅ **Metrics**: Prometheus metrics with /metrics endpoint  
✅ **Logs**: Structured logging with Loki  
✅ **Auto-instrumentation**: Flask automatically instrumented  
✅ **Test Endpoints**: Multiple API endpoints for testing

## Quick Start

### 1. Build and Start Services

```bash
docker-compose up --build -d
```

### 2. Access Dashboards

- **Flask App**: http://localhost:5000
- **Grafana**: http://localhost:3000 (admin/admin)
- **Prometheus**: http://localhost:9090
- **Jaeger UI**: http://localhost:16686
- **App Metrics**: http://localhost:5000/metrics

### 3. Test API Endpoints

```bash
# Basic endpoint
curl http://localhost:5000/

# Get users list
curl http://localhost:5000/api/users

# Get specific user
curl http://localhost:5000/api/users/1

# Create order (nested spans)
curl http://localhost:5000/api/orders

# Slow endpoint (1-3 seconds)
curl http://localhost:5000/api/slow

# Error endpoint (for testing error tracking)
curl http://localhost:5000/api/error

# Health check
curl http://localhost:5000/health

# Prometheus metrics
curl http://localhost:5000/metrics
```

### 4. Generate Traffic (Load Testing)

```bash
# Linux/Mac
for i in {1..100}; do curl http://localhost:5000/; done
for i in {1..50}; do curl http://localhost:5000/api/users; done
for i in {1..30}; do curl http://localhost:5000/api/orders; done

# PowerShell (Windows)
1..100 | ForEach-Object { Invoke-WebRequest -Uri http://localhost:5000/ }
1..50 | ForEach-Object { Invoke-WebRequest -Uri http://localhost:5000/api/users }
1..30 | ForEach-Object { Invoke-WebRequest -Uri http://localhost:5000/api/orders }
```

## Configure Grafana

### 1. Add Data Sources

**Prometheus:**
1. Go to Configuration > Data Sources
2. Add Prometheus
3. URL: `http://prometheus:9090`
4. Click "Save & Test"

**Loki:**
1. Add Loki data source
2. URL: `http://loki:3100`
3. Click "Save & Test"

**Jaeger:**
1. Add Jaeger data source
2. URL: `http://jaeger:16686`
3. Click "Save & Test"

### 2. Create Dashboards

**Metrics Dashboard:**
- Query: `rate(http_requests_total[5m])`
- Query: `histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))`
- Query: `otel_app_requests_total`

**Logs Dashboard:**
- Switch to Loki data source
- Query: `{service="otel-sample-app"}`
- Query: `{container="otel-sample-app"} |= "error"`

## Metrics Available

### Prometheus Metrics (from /metrics endpoint)
- `http_requests_total` - Total HTTP requests by method, endpoint, status
- `http_request_duration_seconds` - Request duration histogram

### OTel Metrics (exported via collector)
- `app.requests.total` - Total requests
- `app.request.duration` - Request duration

## Traces

All endpoints create traces with:
- Automatic Flask instrumentation
- Custom spans for business logic
- Nested spans (e.g., order creation)
- Error tracking with exceptions

View traces in Jaeger UI at http://localhost:16686

## Logs

Logs are collected from:
- Application logs (via OTLP)
- Container logs (via Promtail)

View logs in Grafana using Loki data source.

## Troubleshooting

### Check Service Status
```bash
docker-compose ps
```

### View Logs
```bash
docker-compose logs app
docker-compose logs otel-collector
docker-compose logs prometheus
```

### Restart Services
```bash
docker-compose restart
```

### Clean Up
```bash
docker-compose down -v
```

## Project Structure

```
.
├── app.py                      # Flask application with OTel instrumentation
├── requirement.txt             # Python dependencies
├── Dockerfile                  # Docker image for Flask app
├── docker-compose.yaml         # All services configuration
├── otel-collector-config.yaml  # OTel Collector configuration
├── prometheus.yml              # Prometheus scrape configuration
├── loki-config.yaml           # Loki configuration
└── promtail-config.yaml       # Promtail log shipping config
```

## Key Concepts

### Traces
- **Span**: A single operation (e.g., HTTP request, database query)
- **Trace**: A collection of spans representing a request flow
- View in Jaeger to see request paths and timing

### Metrics
- **Counter**: Cumulative metric (e.g., total requests)
- **Histogram**: Distribution of values (e.g., request duration)
- Query in Prometheus or visualize in Grafana

### Logs
- **Structured logging**: JSON formatted logs
- **OTLP export**: Logs sent to OTel Collector
- **Container logs**: Collected by Promtail
- Query in Grafana using Loki

## Next Steps

1. Create custom Grafana dashboards
2. Set up alerting rules in Prometheus
3. Add more business metrics
4. Configure log retention policies
5. Add authentication to services
