## Day 34/40
# Step-By-Step Guide To Upgrade a Multi Node Kubernetes Cluster With Kubeadm
[Video Link](https://www.youtube.com/watch?v=NtX75Ze47EU)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section, the `kubernetes` cluster wil be update with `kubeadm`. 


Let's assume we have 1 controller-plane with 3 worker nodes, and one is failed for a reason.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/345zt6zp5i39fbznd57s.png)

(Photo from the video)

```
node worker1 drain
```
then
- `workloads` would be evicted.
- `node` is `cordon` and unschedulable.
- The `nginx` pod will schedule in other `node` because it's controlled by `deployment`
- The `mysql` pod and its data and configurations is gone.

 
If we replace or resolve the issues the failed `node`, we need to `uncordon` it to make it shcedulable and ready again.
It will accept new `workload` but not current `workload`. 

---


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rh1i4j3a6lv9d0nnn6ra.png)

(Photo from the video)

For upgrading we cannot skip the minor version and for upgrading we need to upgrade to one next minor version.
For example,at first upgrade `1.28.2` to `1.29.3`, then we can upgrade from `1.29.3` to `1.30.2` and so on.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/eanytvbfdx0nl5l31d14.png)

(Photo from the video)

As a `kubernetes` cluster admin, every month or every couple of months, we need to upgrade the cluster, that's why it's important concept for administration.

**Note** at single time, `kubernetes` only support last 3 minor versions. It means, no new bug fixes or updating the features on that minor version.

>Example:
>
>    kube-apiserver is at `1.31`
>    kubelet is supported at `1.31`, `1.30`, `1.29`, and `1.28`
[source](https://kubernetes.io/releases/version-skew-policy/#kubelet)


Official document for upgrading with `kubeadm`, [here](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
>The upgrade workflow at high level is the following:
>
>    Upgrade a primary control plane node.
>    Upgrade additional control plane nodes.
>    Upgrade worker nodes.


Upgrading strategies:
1. All at once, we have downtime.
2. Rolling update, one by one.
3. Blue Green, upgrading new cluster and transfer workloads from old one.

---

### Upgrade Master node

1. Changing the package repository 
[here](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/change-package-repository/#verifying-if-the-kubernetes-package-repositories-are-used)


2. Determine which version to upgrade to

```
# Find the latest 1.31 version in the list.
# It should look like 1.31.x-*, where x is the latest patch.
sudo apt update
sudo apt-cache madison kubeadm
```
[source](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#determine-which-version-to-upgrade-to)


3. Upgrading control plane nodes 

```
# replace x in 1.31.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.x-*' && \
sudo apt-mark hold kubeadm
```
```
kubeadm version
```


4. Check the upgrade plan

```
kubeadm upgrade plan
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/574cvh0d4f2v2p6w5x6h.png)

(Photo from the video)

```
kubeadm upgrade apply v1.30.2
```

5. Drain the node 

```
kubectl drain <node-to-drain> --ignore-daemonsets
```

6. Upgrade kubelet and kubectl 

```
# replace x in 1.31.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*' && \
sudo apt-mark hold kubelet kubectl

```

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet

```

7. Uncordon the node 

```
# replace <node-to-uncordon> with the name of your node
kubectl uncordon <node-to-uncordon>

```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cudb5dfwshtrx2rvuo70.png)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ib3eqb192gvnqj4xz9o4.png)

(Photos from the video)


### Upgrade worker nodes 

[source](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/#how-it-works)

> How it works
> 
> kubeadm upgrade apply does the following:
> 
>     Checks that your cluster is in an upgradeable state:
> 
>         The API server is reachable
> 
>         All nodes are in the Ready state
> 
>         The control plane is healthy
> 
>     Enforces the version skew policies.
> 
>     Makes sure the control plane images are available or available to pull to the machine.
> 
>     Generates replacements and/or uses user supplied overwrites if component configs require version upgrades.
> 
>     Upgrades the control plane components or rollbacks if any of them fails to come up.
> 
>     Applies the new CoreDNS and kube-proxy manifests and makes sure that all necessary RBAC rules are created.
> 
>     Creates new certificate and key files of the API server and backs up old files if they're about to expire in 180 days.
> 
> kubeadm upgrade node does the following on additional control plane nodes:
> 
>     Fetches the kubeadm ClusterConfiguration from the cluster.
> 
>     Optionally backups the kube-apiserver certificate.
> 
>     Upgrades the static Pod manifests for the control plane components.
> 
>     Upgrades the kubelet configuration for this node.
> 
> kubeadm upgrade node does the following on worker nodes:
> 
>     Fetches the kubeadm ClusterConfiguration from the cluster.
> 
>     Upgrades the kubelet configuration for this node.
> 


[source](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/)

1. Upgrade kubeadm 

```
# replace x in 1.31.x-* with the latest patch version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.31.x-*' && \
sudo apt-mark hold kubeadm

```

2. Call "kubeadm upgrade" 

```
sudo kubeadm upgrade node

```

3. Drain the node 

```
# execute this command on a control plane node
# replace <node-to-drain> with the name of your node you are draining
kubectl drain <node-to-drain> --ignore-daemonsets

```

4. Upgrade kubelet and kubectl 

```
# replace x in 1.31.x-* with the latest patch version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.31.x-*' kubectl='1.31.x-*' && \
sudo apt-mark hold kubelet kubectl

```

```
sudo systemctl daemon-reload
sudo systemctl restart kubelet

```

5. Uncordon the node 

```
# execute this command on a control plane node
# replace <node-to-uncordon> with the name of your node
kubectl uncordon <node-to-uncordon>

```

### Summary

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a884adio7je6bexrciz9.png)


(Photo from the video)











