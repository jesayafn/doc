# Istio Observability

## A. Overview

Istio generates an Observable of telemetry type:

1. Metrics

     Provide set of metrics based on the four "golden signals" (latency, traffic, errors, saturation). This metrics can be accessed on prometheus installed with helm ([Artifact Hub - Prometheus Chart](https://artifacthub.io/packages/helm/prometheus-community/prometheus)) or with following the Istio and Prometheus integration documentation ([Istio Documentation - Prometheus Integration: Option 1: Quick Start](https://istio.io/latest/docs/ops/integrations/prometheus/#option-1-quick-start)).

     Below is an example of Querying on Prometheus to get Istio generated metrics: 
     ![Image 1 - Querying on Prometheus to get Istio generated metrics.](img/i-3-doc_istio-obervability_0.jpeg)

1. Distributed metrics
1. Access Logs

## B. Metrics

Istio generate 3 levels of metrics type:

1. Proxy-level metrics

     Metrics generate by Envoy proxy, a side car of proxy used on istio. All metrics on Istio's Envoy are listed on [Statistic Documentation on Envoy Documentation](https://www.envoyproxy.io/docs/envoy/latest/configuration/configuration)
     > Note: Not all Envoy metrics are available on Istio proxy-level metrics
     * Envoy Proxy metrics showed on Prometheus Metrics Explorer
     ![Envoy Proxy metrics showed on Prometheus Metrics Explorer](img/i-3-doc_istio-obervability_1.jpeg)
     * Querying on Prometheus to get Envoy server uptime in hours
     ![Querying on Prometheus to get Envoy server uptime in hours](img/i-3-doc_istio-obervability_2.jpeg)

1. Service-level metrics

     Service oriented metrics
1. Control plane metrics