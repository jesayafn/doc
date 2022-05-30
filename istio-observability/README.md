# Istio Observability

## A. Overview

### 1. Metrics

Provide set of metrics based on the four "golden signals" (latency, traffic, errors, saturation). This metrics can be accessed on prometheus installed with helm ([Artifact Hub - Prometheus Chart](https://artifacthub.io/packages/helm/prometheus-community/prometheus)) or with following the Istio and Prometheus integration documentation ([Istio Documentation - Prometheus Integration - Option 1: Quick Start](https://istio.io/latest/docs/ops/integrations/prometheus/#option-1-quick-start)).

Below is an example of Querying on Prometheus to get Istio generated metrics:
![Image 1 - Querying on Prometheus to get Istio generated metrics.](img/i-3-doc_istio-obervability_0.jpeg)

### 2. Distributed traces

### 3. Access Logs

## B. Metrics

Istio generate 3 levels of metrics type:

### 1. Proxy-level metrics

Metrics generate by Envoy proxy, a side car of proxy used on istio. All metrics on Istio's Envoy are listed on [Statistic Documentation on Envoy Documentation](https://www.envoyproxy.io/docs/envoy/latest/configuration/configuration)

> Note: Not all Envoy metrics are available on Istio proxy-level metrics

* Envoy Proxy metrics showed on Prometheus Metrics Explorer
![Envoy Proxy metrics showed on Prometheus Metrics Explorer](img/i-3-doc_istio-obervability_1.jpeg)
* Querying on Prometheus to get Envoy server uptime in hours
![Querying on Prometheus to get Envoy server uptime in hours](img/i-3-doc_istio-obervability_2.jpeg)

### 2. Service-level metrics

Service-level metrics is a service oriented metrics to monitor service communication. All Istio service-level metrics are listed on [Istio Standard Metrics Documentation](https://istio.io/latest/docs/reference/config/metrics).

* Querying on Prometheus to get average request duration in milisecond from "productpage" to any destinations except "unknown" every 1 minute.
![Querying on Prometheus to get average request duration in milisecond from "productpage" to any destinations except "unknown" every 1 minute. ](img/i-3-doc_istio-obervability_3.jpeg)

### 3. Control plane metrics

Self-monitoring metrics to monitor istio behavior. All Istio service-level metrics are listed on [Istio Pilot Metrics Documentation](https://istio.io/latest/docs/reference/commands/pilot-discovery/#metrics).

* Querying on Prometheus to get a number of success sidecar injection requests.
![Querying on Prometheus to get a number of success sidecar injection requests.](img/i-3-doc_istio-obervability_4.jpeg)
