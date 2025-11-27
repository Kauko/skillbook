# SLO/SLI Observability

**Description:** Use when defining service level objectives or setting up observability. Defines SLOs/SLIs and generates monitoring code for comprehensive system observability.

## When to Use This Skill

Use this skill when:
- Defining service level objectives for new or existing services
- Setting up observability infrastructure (logging, metrics, tracing)
- Establishing reliability targets and error budgets
- Creating monitoring dashboards and alerts
- Implementing structured logging
- Integrating observability into Clojure applications
- Aligning monitoring with quality requirements

## Core Concepts

### SLO (Service Level Objective)
A target value or range for a service level measured by an SLI. Example: "99.9% of requests complete in under 200ms."

### SLI (Service Level Indicator)
A quantitative measure of service level. Example: Request latency, error rate, availability.

### Error Budget
The allowed amount of unreliability (100% - SLO). Example: 99.9% SLO = 0.1% error budget = ~43 minutes downtime per month.

## Workflow

### 1. Identify Critical User Journeys

Analyze the system to identify key user-facing operations:
- API endpoints
- Background jobs
- Data processing pipelines
- External integrations

For each journey, determine:
- What constitutes success?
- What performance is acceptable?
- What availability is required?

### 2. Define SLIs

Common SLI patterns:

#### Availability SLI
Measures uptime and successful responses.

```yaml
# infrastructure/observability/slis/availability.yaml
sli:
  name: api-availability
  description: Percentage of successful API requests
  type: availability
  measurement:
    numerator: "sum(rate(http_requests_total{status!~'5..'}[5m]))"
    denominator: "sum(rate(http_requests_total[5m]))"
  unit: percent
  good_values: ">= 99.9"
```

#### Latency SLI
Measures response time distribution.

```yaml
# infrastructure/observability/slis/latency.yaml
sli:
  name: api-latency
  description: API request latency at various percentiles
  type: latency
  measurement:
    query: "histogram_quantile({quantile}, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))"
    percentiles:
      - p50: 100   # 50th percentile: 100ms
      - p95: 200   # 95th percentile: 200ms
      - p99: 500   # 99th percentile: 500ms
  unit: milliseconds
```

#### Throughput SLI
Measures request volume handling.

```yaml
# infrastructure/observability/slis/throughput.yaml
sli:
  name: api-throughput
  description: Requests processed per second
  type: throughput
  measurement:
    query: "sum(rate(http_requests_total[5m]))"
    minimum: 100  # At least 100 req/s capacity
  unit: requests_per_second
```

#### Error Rate SLI
Measures failure frequency.

```yaml
# infrastructure/observability/slis/error-rate.yaml
sli:
  name: api-error-rate
  description: Percentage of failed requests
  type: error_rate
  measurement:
    numerator: "sum(rate(http_requests_total{status=~'5..'}[5m]))"
    denominator: "sum(rate(http_requests_total[5m]))"
  unit: percent
  good_values: "<= 0.1"  # Less than 0.1% errors
```

### 3. Define SLOs with Error Budgets

```yaml
# infrastructure/observability/slos/api-service.yaml
service: api-service
owner: platform-team
slos:
  - name: availability-slo
    description: API service should be available 99.9% of the time
    sli: api-availability
    objective:
      target: 99.9
      window: 30d
    error_budget:
      remaining_percent: 0.1  # 0.1% of requests can fail
      remaining_time: 43m     # ~43 minutes downtime per month
    alerts:
      - name: error-budget-burn-fast
        condition: "error_budget_consumed > 10% in 1h"
        severity: critical
        notification: pagerduty
      - name: error-budget-burn-slow
        condition: "error_budget_consumed > 50% in 7d"
        severity: warning
        notification: slack

  - name: latency-slo
    description: 95% of requests complete within 200ms
    sli: api-latency
    objective:
      target: 200
      percentile: 95
      window: 30d
    alerts:
      - name: latency-degradation
        condition: "p95_latency > 200ms for 5m"
        severity: warning
        notification: slack

  - name: error-rate-slo
    description: Error rate below 0.1%
    sli: api-error-rate
    objective:
      target: 0.1
      comparison: "less_than"
      window: 7d
    alerts:
      - name: elevated-error-rate
        condition: "error_rate > 0.1% for 5m"
        severity: critical
        notification: pagerduty
```

### 4. Calculate Error Budgets

```clojure
;; infrastructure/observability/error-budget-calculator.clj
(ns observability.error-budget
  "Calculate error budgets based on SLOs")

(defn calculate-error-budget
  "Calculate error budget for a given SLO over a time window.

  Args:
    slo-percent: Target SLO percentage (e.g., 99.9)
    window-days: Time window in days (e.g., 30)

  Returns:
    Map with error budget details"
  [slo-percent window-days]
  (let [uptime-required (/ slo-percent 100)
        downtime-allowed (- 1 uptime-required)
        minutes-in-window (* window-days 24 60)
        allowed-downtime-minutes (* minutes-in-window downtime-allowed)
        allowed-downtime-seconds (* allowed-downtime-minutes 60)]
    {:slo-percent slo-percent
     :window-days window-days
     :uptime-required-percent (* uptime-required 100)
     :downtime-allowed-percent (* downtime-allowed 100)
     :total-minutes minutes-in-window
     :allowed-downtime-minutes allowed-downtime-minutes
     :allowed-downtime-seconds allowed-downtime-seconds
     :allowed-downtime-hours (/ allowed-downtime-minutes 60)}))

(defn error-budget-remaining
  "Calculate remaining error budget.

  Args:
    total-requests: Total number of requests in window
    failed-requests: Number of failed requests in window
    slo-percent: Target SLO percentage

  Returns:
    Map with remaining budget details"
  [total-requests failed-requests slo-percent]
  (let [actual-success-rate (/ (- total-requests failed-requests) total-requests)
        actual-success-percent (* actual-success-rate 100)
        allowed-failures (* total-requests (- 1 (/ slo-percent 100)))
        remaining-failures (- allowed-failures failed-requests)
        budget-consumed-percent (* (/ failed-requests allowed-failures) 100)]
    {:total-requests total-requests
     :failed-requests failed-requests
     :actual-success-percent actual-success-percent
     :allowed-failures allowed-failures
     :remaining-failures remaining-failures
     :budget-consumed-percent budget-consumed-percent
     :budget-remaining-percent (- 100 budget-consumed-percent)
     :slo-met? (>= actual-success-percent slo-percent)}))

(comment
  ;; Example: 99.9% SLO over 30 days
  (calculate-error-budget 99.9 30)
  ;; => {:slo-percent 99.9
  ;;     :window-days 30
  ;;     :uptime-required-percent 99.9
  ;;     :downtime-allowed-percent 0.1
  ;;     :total-minutes 43200
  ;;     :allowed-downtime-minutes 43.2
  ;;     :allowed-downtime-seconds 2592
  ;;     :allowed-downtime-hours 0.72}

  ;; Example: Check remaining budget
  (error-budget-remaining 1000000 50 99.9)
  ;; => {:total-requests 1000000
  ;;     :failed-requests 50
  ;;     :actual-success-percent 99.995
  ;;     :allowed-failures 1000
  ;;     :remaining-failures 950
  ;;     :budget-consumed-percent 5.0
  ;;     :budget-remaining-percent 95.0
  ;;     :slo-met? true}
  )
```

### 5. Integration with ISO 25010 Quality Requirements

Align SLOs with ISO 25010 quality characteristics from the `iso25010-quality` skill:

```yaml
# infrastructure/observability/quality-slo-mapping.yaml
quality_slo_mapping:
  # Performance Efficiency
  - iso25010_characteristic: performance_efficiency
    sub_characteristic: time_behaviour
    slos:
      - api-latency-slo
      - database-query-latency-slo
    rationale: "Time behaviour measured through latency SLIs"

  # Reliability
  - iso25010_characteristic: reliability
    sub_characteristic: availability
    slos:
      - api-availability-slo
      - database-availability-slo
    rationale: "System availability measured through uptime SLIs"

  - iso25010_characteristic: reliability
    sub_characteristic: fault_tolerance
    slos:
      - api-error-rate-slo
      - retry-success-rate-slo
    rationale: "Fault tolerance measured through error rates"

  # Maintainability
  - iso25010_characteristic: maintainability
    sub_characteristic: analysability
    slos:
      - log-ingestion-slo
      - trace-sampling-slo
    rationale: "System analysability through logging and tracing"
```

### 6. Generate Observability Code

#### Structured Logging (Clojure)

```clojure
;; src/observability/logging.clj
(ns observability.logging
  "Structured logging utilities for observability"
  (:require [jsonista.core :as json]
            [clojure.tools.logging :as log]))

(def object-mapper
  "JSON object mapper for log serialization"
  (json/object-mapper {:pretty false}))

(defn log-structured
  "Log a structured message as JSON.

  Args:
    level: Log level (:info, :warn, :error, :debug)
    event: Event name/type
    context: Map of additional context"
  [level event context]
  (let [log-entry (merge
                   {:timestamp (System/currentTimeMillis)
                    :event event
                    :level (name level)}
                   context)
        json-str (json/write-value-as-string log-entry object-mapper)]
    (case level
      :debug (log/debug json-str)
      :info (log/info json-str)
      :warn (log/warn json-str)
      :error (log/error json-str)
      (log/info json-str))))

(defmacro with-request-context
  "Execute body with request context in logs.

  Automatically includes request-id, user-id, and session-id in all logs."
  [request & body]
  `(let [request-id# (or (get-in ~request [:headers "x-request-id"])
                         (str (java.util.UUID/randomUUID)))
         user-id# (get-in ~request [:session :user-id])
         session-id# (get-in ~request [:session :id])]
     (with-redefs [log-structured
                   (fn [level# event# context#]
                     (log-structured level# event#
                                     (merge {:request-id request-id#
                                             :user-id user-id#
                                             :session-id session-id#}
                                            context#)))]
       ~@body)))

(defn log-request
  "Log HTTP request"
  [request]
  (log-structured :info "http.request"
                  {:method (:request-method request)
                   :uri (:uri request)
                   :query-string (:query-string request)
                   :remote-addr (:remote-addr request)}))

(defn log-response
  "Log HTTP response with duration"
  [response duration-ms]
  (log-structured :info "http.response"
                  {:status (:status response)
                   :duration-ms duration-ms}))

(defn log-error
  "Log error with exception details"
  [event error context]
  (log-structured :error event
                  (merge context
                         {:error-type (type error)
                          :error-message (.getMessage error)
                          :stack-trace (mapv str (.getStackTrace error))})))

(comment
  ;; Example usage
  (log-structured :info "user.login" {:user-id 123 :ip "192.168.1.1"})
  ;; => {"timestamp":1234567890,"event":"user.login","level":"info","user-id":123,"ip":"192.168.1.1"}

  (log-error "database.query.failed"
             (ex-info "Connection timeout" {:query "SELECT * FROM users"})
             {:retry-count 3})
  )
```

#### Metrics (Prometheus/Micrometer)

```clojure
;; src/observability/metrics.clj
(ns observability.metrics
  "Metrics collection for Prometheus"
  (:require [iapetos.core :as prometheus]
            [iapetos.collector.ring :as ring-collector]))

(defonce registry
  "Global Prometheus registry"
  (-> (prometheus/collector-registry)
      (prometheus/register
       (prometheus/counter :http/requests-total
                           {:description "Total HTTP requests"
                            :labels [:method :status :path]})

       (prometheus/histogram :http/request-duration-seconds
                             {:description "HTTP request duration"
                              :labels [:method :path]
                              :buckets [0.01 0.05 0.1 0.25 0.5 1.0 2.5 5.0]})

       (prometheus/gauge :app/active-connections
                         {:description "Number of active connections"})

       (prometheus/counter :db/queries-total
                           {:description "Total database queries"
                            :labels [:operation :table :status]})

       (prometheus/histogram :db/query-duration-seconds
                             {:description "Database query duration"
                              :labels [:operation :table]
                              :buckets [0.001 0.005 0.01 0.05 0.1 0.5 1.0]})

       (prometheus/counter :external/api-calls-total
                           {:description "Total external API calls"
                            :labels [:service :endpoint :status]})

       (prometheus/histogram :external/api-call-duration-seconds
                             {:description "External API call duration"
                              :labels [:service :endpoint]
                              :buckets [0.1 0.5 1.0 2.0 5.0 10.0]}))))

(defn inc-counter
  "Increment a counter metric"
  [metric-key labels]
  (prometheus/inc (prometheus/counter registry metric-key) labels))

(defn observe-histogram
  "Observe a value in histogram"
  [metric-key labels value]
  (prometheus/observe (prometheus/histogram registry metric-key) labels value))

(defn set-gauge
  "Set gauge value"
  [metric-key value]
  (prometheus/set (prometheus/gauge registry metric-key) value))

(defmacro with-timing
  "Time execution and record in histogram"
  [metric-key labels & body]
  `(let [start# (System/nanoTime)
         result# (do ~@body)
         duration# (/ (- (System/nanoTime) start#) 1e9)]
     (observe-histogram ~metric-key ~labels duration#)
     result#))

(defn wrap-metrics
  "Ring middleware to collect HTTP metrics"
  [handler]
  (fn [request]
    (let [start (System/nanoTime)
          response (handler request)
          duration (/ (- (System/nanoTime) start) 1e9)
          method (name (:request-method request))
          path (:uri request)
          status (str (:status response))]

      ;; Record metrics
      (inc-counter :http/requests-total
                   {:method method :status status :path path})
      (observe-histogram :http/request-duration-seconds
                         {:method method :path path}
                         duration)

      response)))

(defn metrics-handler
  "Handler to expose Prometheus metrics"
  [_request]
  {:status 200
   :headers {"Content-Type" "text/plain; version=0.0.4"}
   :body (prometheus/dump-metrics registry)})

(comment
  ;; Example usage
  (inc-counter :http/requests-total
               {:method "GET" :status "200" :path "/api/users"})

  (with-timing :db/query-duration-seconds
               {:operation "select" :table "users"}
    (Thread/sleep 50)
    {:result "data"})
  )
```

#### Distributed Tracing (OpenTelemetry)

```clojure
;; src/observability/tracing.clj
(ns observability.tracing
  "Distributed tracing with OpenTelemetry"
  (:import [io.opentelemetry.api OpenTelemetry]
           [io.opentelemetry.api.trace Span Tracer SpanKind StatusCode]
           [io.opentelemetry.api.common Attributes AttributeKey]
           [io.opentelemetry.sdk OpenTelemetrySdk]
           [io.opentelemetry.sdk.trace SdkTracerProvider]
           [io.opentelemetry.sdk.trace.export BatchSpanProcessor]
           [io.opentelemetry.exporter.otlp.trace OtlpGrpcSpanExporter]))

(defonce ^:private otel-instance (atom nil))

(defn initialize-tracing!
  "Initialize OpenTelemetry tracing.

  Args:
    service-name: Name of the service
    endpoint: OTLP endpoint URL (e.g., http://localhost:4317)"
  [service-name endpoint]
  (let [exporter (-> (OtlpGrpcSpanExporter/builder)
                     (.setEndpoint endpoint)
                     (.build))
        processor (BatchSpanProcessor/builder exporter)
        tracer-provider (-> (SdkTracerProvider/builder)
                            (.addSpanProcessor (.build processor))
                            (.build))
        otel (-> (OpenTelemetrySdk/builder)
                 (.setTracerProvider tracer-provider)
                 (.buildAndRegisterGlobal))]
    (reset! otel-instance otel)
    otel))

(defn get-tracer
  "Get a tracer instance"
  [instrumentation-name]
  (when-let [otel @otel-instance]
    (.getTracer otel instrumentation-name)))

(defmacro with-span
  "Execute body within a traced span.

  Args:
    span-name: Name of the span
    attributes: Map of span attributes
    body: Code to execute"
  [span-name attributes & body]
  `(if-let [tracer# (get-tracer "app")]
     (let [span# (-> (.spanBuilder tracer# ~span-name)
                     (.setSpanKind SpanKind/INTERNAL)
                     (.startSpan))
           scope# (.makeCurrent span#)]
       ;; Set attributes
       (doseq [[k# v#] ~attributes]
         (.setAttribute span# (AttributeKey/stringKey (name k#)) (str v#)))

       (try
         (let [result# (do ~@body)]
           (.setStatus span# StatusCode/OK)
           result#)
         (catch Exception e#
           (.setStatus span# StatusCode/ERROR (.getMessage e#))
           (.recordException span# e#)
           (throw e#))
         (finally
           (.end span#)
           (.close scope#))))
     ;; If tracing not initialized, just execute body
     (do ~@body)))

(defn add-event
  "Add an event to current span"
  [event-name attributes]
  (when-let [span (Span/current)]
    (let [attrs (reduce-kv
                 (fn [acc k v]
                   (.put acc (AttributeKey/stringKey (name k)) (str v))
                   acc)
                 (Attributes/builder)
                 attributes)]
      (.addEvent span event-name (.build attrs)))))

(defn wrap-tracing
  "Ring middleware to create spans for HTTP requests"
  [handler]
  (fn [request]
    (with-span (str (name (:request-method request)) " " (:uri request))
               {:http.method (name (:request-method request))
                :http.url (:uri request)
                :http.client_ip (:remote-addr request)}
      (let [response (handler request)]
        (add-event "response-sent"
                   {:http.status_code (:status response)})
        response))))

(comment
  ;; Initialize tracing
  (initialize-tracing! "my-service" "http://localhost:4317")

  ;; Use in code
  (with-span "process-order"
             {:order-id 123 :user-id 456}
    (Thread/sleep 100)
    (add-event "order-validated" {:amount 99.99})
    (Thread/sleep 50)
    {:status :completed})
  )
```

### 7. Dashboard Specifications

```yaml
# infrastructure/observability/dashboards/api-service.yaml
dashboard:
  title: API Service SLOs
  tags: [slo, api, production]
  refresh: 30s

  variables:
    - name: environment
      type: query
      query: label_values(environment)
      default: production

  rows:
    - title: SLO Overview
      panels:
        - title: Availability SLO
          type: stat
          targets:
            - expr: |
                100 * (
                  sum(rate(http_requests_total{status!~"5..", env="$environment"}[30d]))
                  /
                  sum(rate(http_requests_total{env="$environment"}[30d]))
                )
          thresholds:
            - value: 99.9
              color: green
            - value: 99.5
              color: yellow
            - value: 0
              color: red

        - title: Error Budget Remaining
          type: gauge
          targets:
            - expr: |
                100 - (
                  100 * sum(rate(http_requests_total{status=~"5..", env="$environment"}[30d]))
                  /
                  (0.001 * sum(rate(http_requests_total{env="$environment"}[30d])))
                )
          unit: percent
          min: 0
          max: 100

        - title: P95 Latency
          type: stat
          targets:
            - expr: |
                histogram_quantile(0.95,
                  sum(rate(http_request_duration_seconds_bucket{env="$environment"}[5m])) by (le)
                )
          unit: seconds
          thresholds:
            - value: 0.2
              color: green
            - value: 0.5
              color: yellow
            - value: 1.0
              color: red

    - title: Request Rate and Errors
      panels:
        - title: Request Rate
          type: graph
          targets:
            - expr: sum(rate(http_requests_total{env="$environment"}[5m]))
              legend: Total
            - expr: sum(rate(http_requests_total{status=~"5..", env="$environment"}[5m]))
              legend: Errors

        - title: Error Rate
          type: graph
          targets:
            - expr: |
                100 * (
                  sum(rate(http_requests_total{status=~"5..", env="$environment"}[5m]))
                  /
                  sum(rate(http_requests_total{env="$environment"}[5m]))
                )
          unit: percent

    - title: Latency Distribution
      panels:
        - title: Latency Percentiles
          type: graph
          targets:
            - expr: histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket{env="$environment"}[5m])) by (le))
              legend: P50
            - expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket{env="$environment"}[5m])) by (le))
              legend: P95
            - expr: histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket{env="$environment"}[5m])) by (le))
              legend: P99

        - title: Latency Heatmap
          type: heatmap
          targets:
            - expr: sum(rate(http_request_duration_seconds_bucket{env="$environment"}[5m])) by (le)
```

### 8. Alert Configuration

```yaml
# infrastructure/observability/alerts/slo-alerts.yaml
groups:
  - name: slo-alerts
    interval: 30s
    rules:
      # Fast burn: 10% budget consumed in 1 hour
      - alert: ErrorBudgetBurnRateFast
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[1h]))
            /
            sum(rate(http_requests_total[1h]))
          ) > (0.1 * 0.001)  # 10% of 0.1% error budget
        for: 5m
        labels:
          severity: critical
          slo: availability
        annotations:
          summary: "Fast error budget burn detected"
          description: "10% of monthly error budget consumed in 1 hour"

      # Slow burn: 50% budget consumed in 7 days
      - alert: ErrorBudgetBurnRateSlow
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[7d]))
            /
            sum(rate(http_requests_total[7d]))
          ) > (0.5 * 0.001)  # 50% of 0.1% error budget
        for: 15m
        labels:
          severity: warning
          slo: availability
        annotations:
          summary: "Slow error budget burn detected"
          description: "50% of monthly error budget consumed in 7 days"

      # Latency SLO violation
      - alert: LatencySLOViolation
        expr: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
          ) > 0.2
        for: 5m
        labels:
          severity: warning
          slo: latency
        annotations:
          summary: "P95 latency exceeds SLO"
          description: "P95 latency is {{ $value }}s, exceeding 200ms SLO"

      # Error rate SLO violation
      - alert: ErrorRateSLOViolation
        expr: |
          100 * (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) > 0.1
        for: 5m
        labels:
          severity: critical
          slo: error-rate
        annotations:
          summary: "Error rate exceeds SLO"
          description: "Error rate is {{ $value }}%, exceeding 0.1% SLO"
```

### 9. Document in arc42

Update `vault/arc42/08-crosscutting-concepts.md`:

```markdown
## Observability and Monitoring

### SLO/SLI Framework

#### Service Level Indicators (SLIs)
- **Availability**: Percentage of successful requests (target: 99.9%)
- **Latency**: P95 request duration (target: < 200ms)
- **Error Rate**: Percentage of failed requests (target: < 0.1%)
- **Throughput**: Requests per second (capacity: > 100 rps)

#### Service Level Objectives (SLOs)
| Service | SLO | Window | Error Budget |
|---------|-----|--------|--------------|
| API Service | 99.9% availability | 30 days | 43 minutes |
| API Service | P95 < 200ms | 30 days | 5% requests |
| Database | 99.95% availability | 30 days | 21 minutes |

### Observability Stack

#### Logging
- **Format**: Structured JSON logs
- **Fields**: timestamp, level, event, request-id, user-id, context
- **Storage**: [CloudWatch/Stackdriver/ELK]
- **Retention**: 30 days

#### Metrics
- **System**: Prometheus
- **Format**: OpenMetrics
- **Scrape Interval**: 15s
- **Retention**: 90 days
- **Visualization**: Grafana

#### Tracing
- **System**: OpenTelemetry
- **Backend**: [Jaeger/Tempo/X-Ray]
- **Sampling**: 5% of requests
- **Retention**: 7 days

### Integration with Quality Requirements

Maps to ISO 25010 characteristics:
- **Performance Efficiency** → Latency and throughput SLIs
- **Reliability** → Availability and error rate SLIs
- **Maintainability** → Log coverage and trace sampling

### Alert Strategy

- **Critical**: Page on-call (PagerDuty)
  - Fast error budget burn (10% in 1 hour)
  - Availability SLO violation
  - Error rate SLO violation

- **Warning**: Notify team (Slack)
  - Slow error budget burn (50% in 7 days)
  - Latency SLO violation
  - Resource utilization thresholds
```

### 10. Integration with Clojure Libraries

Common libraries for observability:

```clojure
;; project.clj or deps.edn
{:deps
 {;; Logging
  org.clojure/tools.logging {:mvn/version "1.3.0"}
  ch.qos.logback/logback-classic {:mvn/version "1.4.11"}
  metosin/jsonista {:mvn/version "0.3.7"}

  ;; Metrics
  io.github.clj-commons/iapetos {:mvn/version "0.1.13"}
  io.prometheus/simpleclient_hotspot {:mvn/version "0.16.0"}

  ;; Tracing
  io.opentelemetry/opentelemetry-api {:mvn/version "1.31.0"}
  io.opentelemetry/opentelemetry-sdk {:mvn/version "1.31.0"}
  io.opentelemetry/opentelemetry-exporter-otlp {:mvn/version "1.31.0"}}}
```

## Best Practices

### DO:
- Define SLOs based on user experience, not system metrics
- Use error budgets to balance reliability and velocity
- Start with fewer, critical SLOs (3-5 per service)
- Make SLOs visible to the entire team
- Review and adjust SLOs quarterly
- Automate alert routing based on severity
- Use structured logging everywhere
- Sample traces intelligently (not 100%)
- Tag all metrics with environment and service labels

### DON'T:
- Define SLOs without user journey analysis
- Set overly aggressive SLOs (99.999%)
- Alert on every metric violation
- Mix logs and metrics concerns
- Hard-code observability configuration
- Ignore error budget policy
- Sample logs (keep all logs, sample traces)
- Use different metric naming conventions

## Common Patterns

### Golden Signals (Google SRE)
1. **Latency**: Time to service requests
2. **Traffic**: Demand on the system
3. **Errors**: Rate of failed requests
4. **Saturation**: Resource utilization

### RED Method (Requests)
1. **Rate**: Requests per second
2. **Errors**: Failed requests per second
3. **Duration**: Request latency

### USE Method (Resources)
1. **Utilization**: Percentage resource in use
2. **Saturation**: Resource queue depth
3. **Errors**: Error counts

## Troubleshooting

### High Cardinality Metrics
```clojure
;; BAD: User ID in label creates millions of series
(prometheus/inc (prometheus/counter registry :requests)
                {:user-id user-id})

;; GOOD: Use aggregated labels
(prometheus/inc (prometheus/counter registry :requests)
                {:user-type (get-user-type user-id)})
```

### Log Volume Management
```clojure
;; Use log levels appropriately
(log-structured :debug "cache.hit" {...})      ; Development only
(log-structured :info "user.login" {...})      ; Important events
(log-structured :warn "retry.attempt" {...})   ; Potential issues
(log-structured :error "db.query.failed" {...}) ; Errors
```

## Next Steps

After defining SLOs/SLIs:
1. Implement observability code in services
2. Deploy Prometheus and Grafana
3. Configure alerting rules
4. Create dashboards for each service
5. Set up alert routing (PagerDuty, Slack)
6. Document runbooks for common alerts
7. Review error budget consumption weekly
8. Iterate on SLOs based on data
