## Day 12/40
# Kubernetes Daemonset Explained - Daemonsets, Job and Cronjob in Kubernetes
[Video Link](https://www.youtube.com/watch?v=kvITrySpy_k)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


This section is about `cronjob`, `job` and `daemonset`.
As the definition, `DaemonSets` are `Kubernetes` API objects that allow you to run `Pods` as a `daemon` on each of your `Nodes`. New Nodes that join the `cluster` will automatically start running `Pods` that are part of a `DaemonSet`.

`DaemonSets` are often used to run long-lived background services such as Node monitoring systems and log collection agents. To ensure complete coverage, it’s important that these apps run a `Pod` on every Node in your `cluster`. [source](https://spacelift.io/blog/kubernetes-daemonset#what-is-a-kubernetes-daemonset)

---

#### 1. Create a `daemonset`:
```yaml
apiVersion: apps/v1

kind:  DaemonSet

metadata:
  name: nginx-ds
  labels:
    env: demo

spec:
  template:
    metadata:
      labels:
        env: demo
      name: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
  selector:
    matchLabels:
      env: demo
```

```console
root@localhost:~# kubectl apply -f daemonset.yaml
daemonset.apps/nginx-ds created
root@localhost:~# kubectl get po
NAME             READY   STATUS    RESTARTS   AGE
nginx-ds-mf4gz   1/1     Running   0          11s
nginx-ds-rslrm   1/1     Running   0          11s
root@localhost:~# kubectl get po -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
nginx-ds-mf4gz   1/1     Running   0          19s   10.244.2.9    lucky-luke-worker2   <none>           <none>
nginx-ds-rslrm   1/1     Running   0          19s   10.244.1.12   lucky-luke-worker    <none>           <none>

```
As you can see, our custom `workload` as `daemonset` is running on all `worker` nodes and because it is not a `control-plane` component and the `control-plane` node doesn't tolerate custom `workload`, it doesn't run on `control-plane` node. (but we can change it)

If one of these `pod` removed, it will run another one on that `node`
```console
root@localhost:~# kubectl get po
NAME             READY   STATUS    RESTARTS   AGE
nginx-ds-mf4gz   1/1     Running   0          9m40s
nginx-ds-rslrm   1/1     Running   0          9m40s
root@localhost:~# kubectl delete pod nginx-ds-mf4gz
pod "nginx-ds-mf4gz" deleted
root@localhost:~# kubectl get po
NAME             READY   STATUS    RESTARTS   AGE
nginx-ds-946m4   1/1     Running   0          5s
nginx-ds-rslrm   1/1     Running   0          10m

```
- See all `daemonset` on our `cluster`:
```console
root@localhost:~# kubectl get ds --all-namespaces
NAMESPACE     NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
default       nginx-ds     2         2         2       2            2           <none>                   10m
kube-system   kindnet      3         3         3       3            3           kubernetes.io/os=linux   3d
kube-system   kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   3d
```

---

#### 2. Cronjobs/Jobs
See the [Cronitor](https://crontab.guru/) website for understand how it can be configured for tasks run in specific time.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/h2ivgxcoezldutv92tra.png)
(Photo from the video)






















