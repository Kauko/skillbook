---
name: slo-sli-observability
description: Use when user wants to define reliability targets, set up SLOs/SLIs, calculate error budgets, or configure alerts for service reliability.
requires:
  tools: []
  skills: [iso25010-quality]
---

# SLO/SLI Observability

## Core Concepts

- **SLI** (Indicator): Quantitative measure (latency, error rate, availability)
- **SLO** (Objective): Target value for SLI (e.g., "99.9% success rate")
- **Error Budget**: Allowed unreliability (100% - SLO). 99.9% = 43 min/month downtime

## Workflow

### 1. Identify Critical Journeys

For each user-facing operation:
- What is success?
- What latency is acceptable?
- What availability is required?

### 2. Define SLIs

**Availability:**
```yaml
sli:
  name: api-availability
  type: availability
  numerator: "sum(rate(http_requests_total{status!~'5..'}[5m]))"
  denominator: "sum(rate(http_requests_total[5m]))"
```

**Latency:**
```yaml
sli:
  name: api-latency
  type: latency
  numerator: "sum(rate(http_request_duration_seconds_bucket{le='0.2'}[5m]))"
  denominator: "sum(rate(http_request_duration_seconds_count[5m]))"
```

### 3. Set SLOs

```yaml
slo:
  name: api-reliability
  slis: [api-availability, api-latency]
  target: 99.9
  window: 30d
  error_budget: 0.1  # 43.2 minutes/month
```

### 4. Create Alerts

**Error budget burn rate:**
```yaml
alert:
  name: high-burn-rate
  expr: |
    slo_error_budget_remaining < 0.5
    and
    slo_burn_rate_1h > 14.4  # 1% budget in 1 hour
  severity: critical
```

### 5. Implement in Clojure

```clojure
(require '[metrics.core :as m])

;; Request counter
(m/counter http-requests-total {:labels [:method :path :status]})

;; Latency histogram
(m/histogram http-request-duration {:labels [:method :path]
                                     :buckets [0.01 0.05 0.1 0.2 0.5 1.0]})

;; Wrap handlers
(defn with-metrics [handler]
  (fn [req]
    (let [start (System/nanoTime)
          resp (handler req)
          duration (/ (- (System/nanoTime) start) 1e9)]
      (m/inc! http-requests-total {:method (:request-method req)
                                    :path (:uri req)
                                    :status (:status resp)})
      (m/observe! http-request-duration duration {:method (:request-method req)
                                                   :path (:uri req)})
      resp)))
```

## File Organization

```
infrastructure/observability/
├── slis/           # SLI definitions
├── slos/           # SLO configurations
├── alerts/         # Alert rules
└── dashboards/     # Grafana dashboards
```

## Success Criteria

- [ ] SLIs defined with PromQL queries
- [ ] SLO targets set with error budgets
- [ ] Alert rules configured for burn rate

## Related Skills

- `iso25010-quality` - Quality requirements drive SLOs
- `arc42-docs` - Section 10 documents quality targets
