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











