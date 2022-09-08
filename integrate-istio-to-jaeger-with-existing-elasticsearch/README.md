# Integrate Istio to Jaeger with Existing Elasticsearch

## A. Overview

Prerequisites:

* Istio installed on Kubernetes cluster. [See Istio installation with Istioctl](https://istio.io/latest/docs/setup/install/istioctl/)
* Existing Elasticsearch cluster. On this documentation, will be use [Elasticsearch with Wazuh](https://documentation.wazuh.com/current/deployment-options/elastic-stack/index.html)
* Installed application on kubernetes cluster with Istio side-car injected. On this documentation, will be use [Bookinfo application, an example application from Istio](https://istio.io/latest/docs/examples/bookinfo/)
* Installed Helm. [See Helm installation documentation](https://helm.sh/docs/intro/install/)

## B. Steps

### 1. Setup Jaeger

1. Add Jaeger helm repository

   > **⚠️ Attention: On Kubernetes master node or some node for accessing Kubernetes⚠️**

    ```bash
    helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
    helm repo update
    ```

1. Copy CA certificate from Elasticsearch cluster node and generate trust CA certificates for Jaeger

   > **⚠️ Attention: On Elasticsearch cluster node⚠️**
   >
   > **⚠️ Attention: All commands run with privileges.⚠️**
   >
   > Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

   ```bash
   scp [some-directory/some-ca-certificates-files.crt] [username]@[ip/fqdn-master-or-some-node-for-accessing-k8s-cluster]:~
   ```

1. Generate trust CA certificates for Jaeger

   > **⚠️ Attention: On Kubernetes master node or some node for accessing Kubernetes⚠️**

    ```bash
    sudo apt install openjdk-11-jre-headless
    mv [some-ca-certificates-files.crt] [some-ca-certificates-files.pem]
    keytool -import -trustcacerts -keystore trust.store -storepass [trust-ca-store-password] -alias es-root -file [some-ca-certificates-files.crt]
    kubectl create configmap jaeger-tls --from-file=trust.store --from-file=[some-ca-certificates-files.crt] -n [jaeger-namespace]
    ```

1. Install Jaeger with Helm

   > **⚠️ Attention: On Kubernetes master node or some node for accessing Kubernetes⚠️**

   ```bash
   vi [jaeger-chart-values.yaml]
   ```

   ```yaml
   storage:
     type: elasticsearch
     elasticsearch:
       host: [elasticsearch-ip]
       port: 9200
       scheme: https
       user: [elasticsearch-user-username]
       password: [elasticsearch-user-password]
   provisionDataStore:
     cassandra: false
     elasticsearch: false
   query:
     service:
       type: NodePort
     cmdlineParams:
       es.tls.ca: "/tls/ca-crt.pem"
     extraConfigmapMounts:
       - name: jaeger-tls
         mountPath: /tls
         subPath: ""
         configMap: jaeger-tls
         readOnly: true
   collector:
     service:
       zipkin:
         port: 9411
     cmdlineParams:
       es.tls.ca: "/tls/ca-crt.pem"
     extraConfigmapMounts:
       - name: jaeger-tls
         mountPath: /tls
         subPath: ""
         configMap: jaeger-tls
         readOnly: true
   spark:
     enabled: true
     cmdlineParams:
       java.opts: "-Djavax.net.ssl.trustStore=/tls/trust.store -Djavax.net.ssl.trustStorePassword=[trust-ca-store-password]"
     extraConfigmapMounts:
       - name: jaeger-tls
         mountPath: /tls
         subPath: ""
         configMap: jaeger-tls
         readOnly: true
   ```

   ```bash
   helm install jaeger jaegertracing/jaeger -n [jaeger-namespace] -f [jaeger-chart-values.yaml] --create-namespace
   ```

1. Configure Istio

   > **⚠️ Attention: On Kubernetes master node or some node for accessing Kubernetes⚠️**

   ```bash
   vi [istio-config.yaml]
   ```

   ```yaml
   apiVersion: install.istio.io/v1alpha1
   kind: IstioOperator
   spec:
     meshConfig:
       enableTracing: true
       defaultConfig:
         tracing:
           sampling: [number-of-sampling/10000-requests]
           zipkin:
             address: "[fqdn-of-jaeger-collector-service]:9411"
   ```

   ```bash
   istioctl install -f [istio-config.yaml]
   kubectl rollout restart all -n bookinfo
   kubectl rollout restart -n istio-system deployment.apps/istio-ingressgateway
   ```
