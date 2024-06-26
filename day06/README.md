## Day 6/40
# Kubernetes Multi Node Cluster Setup Step By Step - Kind
[Video Link](https://www.youtube.com/watch?v=RORhczcOrWs)
@piyushsachdeva 

There are many ways to install the `Kubernetes` such as installing with `Minikube`, `MicroK8s`, `K3s` and `Kubeadm`, but in this section, we're going to install it with `Kind` cluster.
Read More: [Link1](https://spacelift.io/blog/install-kubernetes), [Link2](https://itnext.io/kubernetes-installation-methods-the-complete-guide-1036c860a2b3)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ovrwdu3wm7eaqk9p639y.png)

[kind](https://kind.sigs.k8s.io/) is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

## Installation Process

### 1. Prerequisite
Golang is needed at first
```console
root@localhost:~# apt install golang -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
golang is already the newest version (2:1.18~0ubuntu2).
0 upgraded, 0 newly installed, 0 to remove and 22 not upgraded.

```

### 2. Download `kind` on linux
[Installation Guid](https://kind.sigs.k8s.io/docs/user/quick-start/)
```console

root@localhost:~# [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    98  100    98    0     0   1080      0 --:--:-- --:--:-- --:--:--  1088
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 6381k  100 6381k    0     0  6188k      0  0:00:01  0:00:01 --:--:-- 6188k
root@localhost:~# chmod +x kind
root@localhost:~# mv kind /usr/local/bin/kind
root@localhost:~# kind --version
kind version 0.23.0

```

### 3. Installing kubectl
[Installation Guid](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
```console
root@localhost:~# curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   138  100   138    0     0    752      0 --:--:-- --:--:-- --:--:--   754
100 49.0M  100 49.0M    0     0  53.0M      0 --:--:-- --:--:-- --:--:-- 53.0M
root@localhost:~# curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   138  100   138    0     0    728      0 --:--:-- --:--:-- --:--:--   730
100    64  100    64    0     0    240      0 --:--:-- --:--:-- --:--:--   240
root@localhost:~# echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
kubectl: OK
root@localhost:~# sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
root@localhost:~# kubectl version
Client Version: v1.30.2
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.29.4

```

### 4. Creating a Cluster
Please read the [Release Notes](https://github.com/kubernetes-sigs/kind/releases)
```console
root@localhost:~# kind create cluster --image kindest/node:v1.29.4@sha256:3abb816a5b1061fb15c6e9e60856ec40d56b7b52bcea5f5f1350bc6e2320b6f8 --name jolly
Creating cluster "jolly" ...
 ✓ Ensuring node image (kindest/node:v1.29.4) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-jolly"
You can now use your cluster with:

kubectl cluster-info --context kind-jolly

Thanks for using kind! 😊

root@localhost:~# kubectl cluster-info --context kind-jolly
Kubernetes control plane is running at https://127.0.0.1:36827
CoreDNS is running at https://127.0.0.1:36827/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```













