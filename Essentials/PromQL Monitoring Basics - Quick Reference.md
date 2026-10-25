# PromQL Monitoring Basics - Quick Reference

## 1) Monitoring fundamentals

Monitoring is the practice of collecting numerical measurements from systems over time so you can understand behavior, detect problems, and make decisions.

The core building blocks are:

- Metrics: the numbers being collected
- Time series: a metric sampled over time
- Labels: metadata that identifies dimensions like service, instance, endpoint, region, status
- Dashboards: visual views of metrics over time
- Alerts: rules that notify you when a metric crosses a threshold

---

## 2) What is a metric?

A metric is a measurement about a system.

Examples:

- CPU usage
- Memory usage
- HTTP requests per second
- Request latency
- Error counts

Common metric families:

- Counter: only increases over time
  - Example: total requests handled
- Gauge: can go up or down
  - Example: current memory usage or active connections
- Histogram: captures distributions of values
  - Example: HTTP request durations grouped into buckets
- Summary: also captures distribution summaries, often used for quantiles

---

## 3) What is a time series?

A time series is a stream of values for a metric over time.

Prometheus stores metrics as time series, typically identified by:

- metric name
- labels
- timestamp/value samples

Example conceptually:

- Metric: `http_requests_total`
- Labels: `method="GET"`, `status="200"`, `instance="api-1"`
- Samples: values at each timestamp

This means Prometheus can answer:

- What is the value now?
- What was the value 5 minutes ago?
- How quickly is it increasing or decreasing?

---

## 4) What are labels?

Labels are key/value pairs attached to a metric.

Examples:

- `job="api"`
- `instance="10.0.0.6:9090"`
- `environment="prod"`
- `status="500"`
- `endpoint="/login"`

Labels let you filter and aggregate metrics by dimensions.

Examples:

- only API service metrics
- only failed requests
- only one instance
- only one endpoint

This is what makes PromQL powerful: you can ask questions across multiple dimensions.

---

## 5) Aggregation

Monitoring often requires combining or summarizing multiple series.

Common aggregation functions:

- `sum()`: total across matching series
- `avg()`: average across matching series
- `min()`: minimum value
- `max()`: maximum value
- `count()`: number of matching series

Example:

- `sum by (job) (rate(http_requests_total[5m]))`

This means:

- take the request rate over the last 5 minutes
- filter by matching series
- group by the `job` label
- sum values for each job

---

## 6) Rate and change over time

Most operational questions are about change over time, not just a single point-in-time value.

This is why PromQL heavily uses:

- `rate()`
- `irate()`
- `increase()`

### Why this matters

A counter like `http_requests_total` keeps increasing over time. Looking at the raw number alone doesn’t tell you throughput.

Instead, you compute how fast it increases:

- requests per second
- percentage increase over time
- change during a specific interval

Example:

- `rate(http_requests_total[5m])`

This means:

- take the request counter
- compute the per-second increase over the last 5 minutes

---

## 7) Dashboards

Dashboards visualize metrics over time so humans can understand system behavior.

Common panels:

- request rate
- error rate
- latency
- CPU
- memory
- saturation
- queue depth

Dashboards answer questions like:

- What is happening right now?
- Is the service trending up or down?
- Did performance change after a deploy?

PromQL is typically used behind the panels shown in dashboards.

---

## 8) Alerts

Alerts turn metrics into notifications.

Typical alert conditions:

- CPU usage above 90% for 5 minutes
- Error rate above 5% for 10 minutes
- p95 latency above 500ms
- Queue depth growing continuously

These are usually implemented as PromQL expressions that evaluate to true/false.

---

## 9) A simple mental model

Think of monitoring like this:

- Metrics = the numbers
- Time series = those numbers over time
- Labels = the dimensions
- PromQL = the query language to ask questions
- Dashboards = visual answers
- Alerts = automatic notifications

---

## 10) PromQL examples that make the idea concrete

### Request rate

```promql
rate(http_requests_total[5m])
```

Shows requests per second over the last 5 minutes.

### Requests with 5xx status

```promql
rate(http_requests_total{status=~"5.."}[5m])
```

Shows error request rate for 5xx responses.

### Sum by endpoint

```promql
sum by (endpoint) (rate(http_requests_total[5m]))
```

Shows total request rate grouped by endpoint.

### Average CPU usage

```promql
avg by (instance) (rate(node_cpu_seconds_total{mode!="idle"}[5m])) * 100
```

Shows CPU usage percentage per instance.

---

## 11) What to learn next

If you want to understand PromQL more easily, study these in order:

1. Metrics and metric types
2. Time series
3. Labels and selectors
4. Rate and change over time
5. Aggregation and grouping
6. Histograms for latency
7. Dashboards and alerts

---

## 12) Beginner questions to practice

Try answering these in plain English:

- What is the request rate for the API service?
- Which service has the highest error rate?
- Which endpoint is most heavily used?
- What is the p95 latency?
- Which instance is consuming the most CPU or memory?
- Did the rate spike after a deployment?

These are exactly the kinds of questions PromQL is designed to answer.

---

## 13) Summary

Before learning PromQL deeply, it helps to know that:

- monitoring is about collecting metrics over time
- time series are measured values across timestamps
- labels let you slice data by service, endpoint, instance, or status
- `rate()` and aggregations are central to understanding behavior
- dashboards and alerts are the practical outputs of all this data

Once this mental model is clear, PromQL becomes much easier to read and reason about.
