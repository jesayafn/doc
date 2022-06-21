# Install Kubernetes on Red Hat Enterprise Linux 8 with kubeadm

## A. Overview

Prerequisites

* Node: 2 or more
* Processor: 2 cores or more on every node
* Memory: 2 GB or more on every node
* Storage: 20 GB or more on every node
* Internet Connection on every node

## B. Steps

1. Install CRI-O container runtime

   > **⚠️ Attention: On every node⚠️**
   >
   > **⚠️ Attention: All commands run with privileges.⚠️**
   >
   > Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

    ```bash
    VERSION=[CRI-O version]
    curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable.repo https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/CentOS_8/devel:kubic:libcontainers:stable.repo
    curl -L -o /etc/yum.repos.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/CentOS_8/devel:kubic:libcontainers:stable:cri-o:$VERSION.repo

    yum install cri-o

    yum update runc
    ```

1. Configure networking for CRI-O runtime

   > **⚠️ Attention: On every node⚠️**
   >
   > **⚠️ Attention: All commands run with privileges.⚠️**
   >
   > Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

    ```bash
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF

    sudo modprobe overlay
    sudo modprobe br_netfilter

    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

    # Apply sysctl params without reboot
    sudo sysctl --system
    ```

1. Configure firewall

   > **⚠️ Attention: All commands run with privileges.⚠️**
   >
   > Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

    See [Ports and Protocols list for Kubernetes](https://kubernetes.io/docs/reference/ports-and-protocols/)
   * For control plane node

     ```bash
     firewall-cmd --permanent --add-port=6443/tcp
     firewall-cmd --permanent --add-port=2379-2380/tcp
     firewall-cmd --permanent --add-port=10250/tcp
     firewall-cmd --permanent --add-port=10259/tcp
     firewall-cmd --permanent --add-port=10257/tcp
     firewall-cmd --reload
     ```

   * For worker node

      ```bash
      firewall-cmd --permanent --add-port=10250/tcp
      firewall-cmd --permanent --add-port=30000-32767/tcp
      firewall-cmd --reload
      ```

1. Disable SELinux

   > **⚠️ Attention: On every node⚠️**
   >
   > **⚠️ Attention: All commands run with privileges.⚠️**
   >
   > Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

   ```bash
   vi /etc/selinux/config
   ```

   ```config
   # This file controls the state of SELinux on the system.
   # SELINUX= can take one of these three values:
   #     enforcing - SELinux security policy is enforced.
   #     permissive - SELinux prints warnings instead of enforcing.
   #     disabled - No SELinux policy is loaded.
   SELINUX=permissive
   ...output ommited...
   ```

   ```bash
   reboot
   ```

1. Install and setup kubeadm, kubelet and kubectl

   > **⚠️ Attention: On every node⚠️**
   >
   > **⚠️ Attention: All commands run with privileges.⚠️**
   >
   > Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

   ```bash
   cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
   enabled=1
   gpgcheck=1
   repo_gpgcheck=1
   gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
   exclude=kubelet kubeadm kubectl
   EOF

   yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

   systemctl enable --now kubelet
   ```

1. Initialize Kubernetes Cluster

   > **⚠️ Attention: On control plane node⚠️**
   >
   > **⚠️ Attention: All commands run with privileges.⚠️**
   >
   > Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

   ```bash
   kubeadm init --control-plane-endpoint [control-plane-ip-or-hostname]
   ```

1. Joining worker node

   > **⚠️ Attention: On worker node⚠️**
   >
   > **⚠️ Attention: All commands run with privileges.⚠️**
   >
   > Run `sudo -i` before run any steps in this section and for exit from `sudo -i` session, run `exit` command or Ctrl+D

   ```bash
   kubeadm token create --print-join-command
   kubeadm join 10.8.60.130:6443 --token [some-token] --discovery-token-ca-cert-hash [some-hash] --control-plane
   ```

1. Configure kubectl

   > **⚠️ Attention: On control plane node⚠️**
   >
   > **⚠️ Attention: Command run as regular user with sudoers⚠️**

   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown [user-username]:[user-group] $HOME/.kube/config
   ```
