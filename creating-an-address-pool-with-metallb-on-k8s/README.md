# Creating an Address Pool with MetalLB on Kubernetes

## A. Overview

Prerequisites:

* Existing Kubernetes cluster. You can use MicroK8s or other Kubernetes distributions.
* Helm installed on master node or node for accessing Kubernetes cluster. See ["How to Install Helm" Documentation](https://helm.sh/docs/intro/install)

## B. Steps

1. Adding MetalLB chart repository

   ```bash
   helm repo add metallb https://metallb.github.io/metallb
   helm repo update
   ```

2. Installing MetalLB on Kubernetes cluster

   ```bash
   helm install [chart-name] metallb/metallb -n [namespace-for-metallb] --create-namespace
   ```

3. Configuring address pool for MetalLB

   ```bash
   vi address-pool.yaml
   ```

   ```yaml
   apiVersion: metallb.io/v1alpha1
   kind: AddressPool
   metadata:
     namespace: [namespace-for-metallb]
     name: [AddressPool-name]
   spec:
     protocol: layer2
     addresses:
     - [IP-range]
   ```

   ```bash
   kubectl apply -f address-pool.yaml
   ```
