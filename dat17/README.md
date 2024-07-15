## Day 17/40
# Kubernetes Autoscaling | HPA Vs VPA
[Video Link](https://www.youtube.com/watch?v=afUL5jGoLx0)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section, we're going to explain `Auto Scaling`, `HPA` and `VPA`.
Scaling is changing your workloads to meet the demand and it can be manually or automatically.

Three common solutions for scaling applications in Kubernetes environments are:
1. **Horizontal Pod Autoscaler (HPA)**: Automatically adds or removes for example pod replicas.
2. **Vertical Pod Autoscaler (VPA)**: Automatically adds or adjusts resources for example CPU and memory reservations for your pods.
3. **Cluster Autoscaler**: Automatically adds or removes nodes in a cluster based on all pods’ requested resources.

[source](https://spot.io/resources/kubernetes-autoscaling/3-methods-and-how-to-make-them-great/)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c6096xupm558hhqhvoqq.png)
(Photo from the video)

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s3727vmkvqjx9zuhd9v7.png)
(Photo from the video)

There are another types of `autoscaling` for instance:
- Event based autoscaling ([KEDA](https://www.cncf.io/projects/keda/))
- Cron/Schedule based autoscaling

---
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b02aovgy8xd6h0hxrsu7.png)
[Image Source](https://www.cncf.io/blog/2019/10/29/kubernetes-autoscaling-101-cluster-autoscaler-horizontal-autoscaler-and-vertical-pod-autoscaler/)

---
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fxev4q8o5kvu55aldgwv.png)
[Image Source](https://www.cncf.io/blog/2019/10/29/kubernetes-autoscaling-101-cluster-autoscaler-horizontal-autoscaler-and-vertical-pod-autoscaler/)

---
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iahk3cxoqczcg4oy164c.png)
[Image Source](https://www.cncf.io/blog/2019/10/29/kubernetes-autoscaling-101-cluster-autoscaler-horizontal-autoscaler-and-vertical-pod-autoscaler/)

---

### Demo
For doing some hands-on we need the `metrics-server` pod is running.
```console
root@localhost:~# kubectl get pods -n kube-system
NAME                                               READY   STATUS    RESTARTS      AGE
coredns-7db6d8ff4d-bftnd                           1/1     Running   1 (13d ago)   14d
coredns-7db6d8ff4d-zs54d                           1/1     Running   1 (13d ago)   14d
etcd-lucky-luke-control-plane                      1/1     Running   1 (13d ago)   14d
kindnet-fbwgj                                      1/1     Running   1 (13d ago)   14d
kindnet-hxb7v                                      1/1     Running   1 (13d ago)   14d
kindnet-kh5s6                                      1/1     Running   1 (13d ago)   14d
kube-apiserver-lucky-luke-control-plane            1/1     Running   1 (13d ago)   14d
kube-controller-manager-lucky-luke-control-plane   1/1     Running   1 (13d ago)   14d
kube-proxy-42h2f                                   1/1     Running   1 (13d ago)   14d
kube-proxy-dhzrs                                   1/1     Running   1 (13d ago)   14d
kube-proxy-rlzwk                                   1/1     Running   1 (13d ago)   14d
kube-scheduler-lucky-luke-control-plane            1/1     Running   1 (13d ago)   14d
metrics-server-55677cdb4c-t5wrw                    1/1     Running   0             108s

```

