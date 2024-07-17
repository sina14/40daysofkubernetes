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
For doing some hands-on demo, we need the `metrics-server` pod is running.
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

- The deployment yaml file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
  labels:
    run: php-apache
spec:
  ports:
  - port: 80
  selector:
    run: php-apache
```
It implements 2 workloads in one yaml file, one for `deployment` and one for `service` with 3 `-` between them.
Let's run them
```console
root@localhost:~# kubectl apply -f day17-deploy.yaml
deployment.apps/php-apache created
service/php-apache created
root@localhost:~# kubectl get po,deploy,svc
NAME                              READY   STATUS    RESTARTS   AGE
pod/php-apache-678865dd57-rlgqw   1/1     Running   0          67s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/php-apache   1/1     1            1           67s

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   14d
service/php-apache   ClusterIP   10.96.250.6   <none>        80/TCP    67s
```

We define 50% threshold for cpu usage for the `deployment` and minimum and maximum `replica` for it.
```console
root@localhost:~# kubectl autoscale deploy php-apache --cpu-percent=50 --min=1 --max=10
horizontalpodautoscaler.autoscaling/php-apache autoscaled
root@localhost:~# kubect get hpa
kubect: command not found
root@localhost:~# kubectl get hpa
NAME         REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
php-apache   Deployment/php-apache   cpu: 0%/50%   1         10        1          17s
```

Now, we generate some loads on the `deployment` and see what will be happened.
```console
root@localhost:~# kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache; done"
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!
```

```console
root@localhost:~# kubectl get po,deploy,svc,hpa
NAME                              READY   STATUS    RESTARTS   AGE
pod/load-generator                1/1     Running   0          9s
pod/php-apache-678865dd57-rlgqw   1/1     Running   0          13m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/php-apache   1/1     1            1           13m

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   14d
service/php-apache   ClusterIP   10.96.250.6   <none>        80/TCP    13m

NAME                                             REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/php-apache   Deployment/php-apache   cpu: 0%/50%   1         10        1          6m45s
```

```console
root@localhost:~# kubectl get po,deploy,svc,hpa
NAME                              READY   STATUS    RESTARTS   AGE
pod/load-generator                1/1     Running   0          30s
pod/php-apache-678865dd57-g4gl6   1/1     Running   0          6s
pod/php-apache-678865dd57-rlgqw   1/1     Running   0          13m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/php-apache   2/2     2            2           13m

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   14d
service/php-apache   ClusterIP   10.96.250.6   <none>        80/TCP    13m

NAME                                             REFERENCE               TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/php-apache   Deployment/php-apache   cpu: 61%/50%   1         10        1          7m6s
```

```console
root@localhost:~# kubectl get po,deploy,svc,hpa
NAME                              READY   STATUS    RESTARTS   AGE
pod/load-generator                1/1     Running   0          99s
pod/php-apache-678865dd57-g4gl6   1/1     Running   0          75s
pod/php-apache-678865dd57-gfjpb   1/1     Running   0          15s
pod/php-apache-678865dd57-gh5fl   1/1     Running   0          60s
pod/php-apache-678865dd57-hkn4b   1/1     Running   0          60s
pod/php-apache-678865dd57-k4g55   1/1     Running   0          45s
pod/php-apache-678865dd57-rlgqw   1/1     Running   0          15m
pod/php-apache-678865dd57-td5ht   1/1     Running   0          45s
pod/php-apache-678865dd57-w69s2   1/1     Running   0          15s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/php-apache   8/8     8            8           15m

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   14d
service/php-apache   ClusterIP   10.96.250.6   <none>        80/TCP    15m

NAME                                             REFERENCE               TARGETS        MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/php-apache   Deployment/php-apache   cpu: 59%/50%   1         10        6          8m15s
```
And when we stop the loads, the replicas start to decreasing
```
OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!OK!^Cpod "load-generator" deleted
pod default/load-generator terminated (Error)
```

```console
root@localhost:~# kubectl get po,deploy,svc,hpa
NAME                              READY   STATUS    RESTARTS   AGE
pod/php-apache-678865dd57-rlgqw   1/1     Running   0          21m

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/php-apache   1/1     1            1           21m

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP   14d
service/php-apache   ClusterIP   10.96.250.6   <none>        80/TCP    21m

NAME                                             REFERENCE               TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/php-apache   Deployment/php-apache   cpu: 0%/50%   1         10        2          14m

```



