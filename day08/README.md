## Day 8/40
# Kubernetes Deployment, Replication Controller and ReplicaSet Explained
[Video Link](https://www.youtube.com/watch?v=oe2zjRb51F0)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)


We're going to learn about `deployment` and `replicaset`.
Assume there's a `pod` containing a `nginx` container running and because of an issue, the `pod` is stopped and crashed.
Actually in the real world, we don't tolerate stopping services, because we have some important customer and we are committed to our `SLA` with them and need to back to up and running state.
So, to overcome that type of problem we implement `HA` with having at least a copy of our app beside.

Let's say we are going to use `replication controller` that's manage by `controller manager`.
**Note** there's difference between `replication controller` and `replicaset`.

`ReplicationController` It creates a certain number of identical pods, and if a pod fails, replaces it. It also lets you update several pods or delete pods with one command. It's responsible for load balancing the traffic between replicas. replicas can be deploy on different nodes.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o3qht34gen8xpozo9y1v.png)
(Photo from the video)


### 1. Create a `ReplicationController`
```yaml
---
apiVersion: v1

kind: ReplicationController

metadata:
  name: nginx-rc
  labels:
    env: demo

spec:
  template:
    metadata:
      name: nginx-pod
      labels:
        env: demo-pod
        type: frontend

    spec:
      containers:
      - name: nginx-container
        image: nginx
        ports:
          - containerPort: 80

  replicas: 3
```

```console
root@localhost:~# kubectl apply -f day08-rc.yaml
replicationcontroller/nginx-rc created
root@localhost:~# kubectl get rc
NAME       DESIRED   CURRENT   READY   AGE
nginx-rc   3         3         3       9s
root@localhost:~# kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
nginx-rc-5x9bx   1/1     Running   0          17s
nginx-rc-ljwqs   1/1     Running   0          17s
nginx-rc-rsvfg   1/1     Running   0          17s
root@localhost:~# kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE    IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-rc-5x9bx   1/1     Running   0          3m2s   10.244.1.3   lucky-luke-worker    <none>           <none>
nginx-rc-ljwqs   1/1     Running   0          3m2s   10.244.2.4   lucky-luke-worker2   <none>           <none>
nginx-rc-rsvfg   1/1     Running   0          3m2s   10.244.2.5   lucky-luke-worker2   <none>           <none>

```
`ReplicaSet` is the new version of `ReplicationController` and is declared in the same way, except it has more options for selectors.

```yaml
---
apiVersion: apps/v1

kind: ReplicaSet

metadata:
  name: nginx-rs
  labels:
    env: demo

spec:
  template:
    metadata:
      name: nginx-pod
      labels:
        env: demo-pod
        type: frontend

    spec:
      containers:
      - name: nginx-container
        image: nginx
        ports:
          - containerPort: 80

  replicas: 3
  selector:
    matchLabels:
      env: demo-pod
```

```console
root@localhost:~# kubectl apply -f day08-rs.yaml
replicaset.apps/nginx-rs created
root@localhost:~# kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   3         3         3       8s
root@localhost:~# kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
nginx-rc-5x9bx   1/1     Running   0          14m
nginx-rc-ljwqs   1/1     Running   0          14m
nginx-rc-rsvfg   1/1     Running   0          14m
nginx-rs-fw8xc   1/1     Running   0          13s
nginx-rs-mn2wp   1/1     Running   0          13s
nginx-rs-wvt7n   1/1     Running   0          13s
root@localhost:~# kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-rc-5x9bx   1/1     Running   0          14m   10.244.1.3   lucky-luke-worker    <none>           <none>
nginx-rc-ljwqs   1/1     Running   0          14m   10.244.2.4   lucky-luke-worker2   <none>           <none>
nginx-rc-rsvfg   1/1     Running   0          14m   10.244.2.5   lucky-luke-worker2   <none>           <none>
nginx-rs-fw8xc   1/1     Running   0          19s   10.244.2.6   lucky-luke-worker2   <none>           <none>
nginx-rs-mn2wp   1/1     Running   0          19s   10.244.1.5   lucky-luke-worker    <none>           <none>
nginx-rs-wvt7n   1/1     Running   0          19s   10.244.1.4   lucky-luke-worker    <none>           <none>
root@localhost:~# kubectl delete -f day08-rc.yaml
replicationcontroller "nginx-rc" deleted
root@localhost:~# kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE     IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-rs-fw8xc   1/1     Running   0          3m19s   10.244.2.6   lucky-luke-worker2   <none>           <none>
nginx-rs-mn2wp   1/1     Running   0          3m19s   10.244.1.5   lucky-luke-worker    <none>           <none>
nginx-rs-wvt7n   1/1     Running   0          3m19s   10.244.1.4   lucky-luke-worker    <none>           <none>

```
Live edit `ReplicaSet` and change the `replicas` to 5.
```console
root@localhost:~# kubectl edit rs nginx-rs
replicaset.apps/nginx-rs edited
root@localhost:~# kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   5         5         5       5m59s
root@localhost:~# kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-5bxcm   1/1     Running   0          11s
nginx-rs-fw8xc   1/1     Running   0          6m6s
nginx-rs-jv4sd   1/1     Running   0          11s
nginx-rs-mn2wp   1/1     Running   0          6m6s
nginx-rs-wvt7n   1/1     Running   0          6m6s
root@localhost:~# kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE     IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-rs-5bxcm   1/1     Running   0          18s     10.244.1.6   lucky-luke-worker    <none>           <none>
nginx-rs-fw8xc   1/1     Running   0          6m13s   10.244.2.6   lucky-luke-worker2   <none>           <none>
nginx-rs-jv4sd   1/1     Running   0          18s     10.244.2.7   lucky-luke-worker2   <none>           <none>
nginx-rs-mn2wp   1/1     Running   0          6m13s   10.244.1.5   lucky-luke-worker    <none>           <none>
nginx-rs-wvt7n   1/1     Running   0          6m13s   10.244.1.4   lucky-luke-worker    <none>           <none>
```
We can use 2 other ways to increase/decrease the `replicas`. One by editing the manifest and then apply with `kubectl` command, and other is what you can see below:
```console
root@localhost:~# kubectl scale --replicas=10 rs/nginx-rs
replicaset.apps/nginx-rs scaled
root@localhost:~# kubectl get rs
NAME       DESIRED   CURRENT   READY   AGE
nginx-rs   10        10        10      11m
root@localhost:~# kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
nginx-rs-5bxcm   1/1     Running   0          5m29s
nginx-rs-7n6vt   1/1     Running   0          6s
nginx-rs-9qk4m   1/1     Running   0          6s
nginx-rs-dwtcb   1/1     Running   0          6s
nginx-rs-fw8xc   1/1     Running   0          11m
nginx-rs-hbxb8   1/1     Running   0          6s
nginx-rs-jv4sd   1/1     Running   0          5m29s
nginx-rs-kh4gq   1/1     Running   0          6s
nginx-rs-mn2wp   1/1     Running   0          11m
nginx-rs-wvt7n   1/1     Running   0          11m
root@localhost:~# kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
nginx-rs-5bxcm   1/1     Running   0          5m41s   10.244.1.6    lucky-luke-worker    <none>           <none>
nginx-rs-7n6vt   1/1     Running   0          18s     10.244.1.8    lucky-luke-worker    <none>           <none>
nginx-rs-9qk4m   1/1     Running   0          18s     10.244.2.9    lucky-luke-worker2   <none>           <none>
nginx-rs-dwtcb   1/1     Running   0          18s     10.244.2.8    lucky-luke-worker2   <none>           <none>
nginx-rs-fw8xc   1/1     Running   0          11m     10.244.2.6    lucky-luke-worker2   <none>           <none>
nginx-rs-hbxb8   1/1     Running   0          18s     10.244.2.10   lucky-luke-worker2   <none>           <none>
nginx-rs-jv4sd   1/1     Running   0          5m41s   10.244.2.7    lucky-luke-worker2   <none>           <none>
nginx-rs-kh4gq   1/1     Running   0          18s     10.244.1.7    lucky-luke-worker    <none>           <none>
nginx-rs-mn2wp   1/1     Running   0          11m     10.244.1.5    lucky-luke-worker    <none>           <none>
nginx-rs-wvt7n   1/1     Running   0          11m     10.244.1.4    lucky-luke-worker    <none>           <none>

```

---

### 2. Deployment
It provides some additional functionalities to the `ReplicaSet`. As a user, we create the `deployment` and it creates the `ReplicaSet` and then the `ReplicaSet` creates `pod`.
So, they are part of the `deployment`
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nmdnpug6eh3a6udt2w3h.png)
(Photo from the video)
In `RollingUpdate` we can update the version of our app in almost zero downtime, which is very important to us and our customers. Also we can do the `RollBack` the changes we made easily.

Let's do it
```yaml
---
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-deploy
  labels:
    env: demo

spec:
  template:
    metadata:
      name: nginx-pod
      labels:
        env: demo-pod
        type: frontend

    spec:
      containers:
      - name: nginx-container
        image: nginx
        ports:
          - containerPort: 80

  replicas: 3
  selector:
    matchLabels:
      env: demo-pod
```

```console
root@localhost:~# kubectl apply -f day08-deploy.yaml
deployment.apps/nginx-deploy created
root@localhost:~# kubectl get node,deploy,rs,pod
NAME                            STATUS   ROLES           AGE     VERSION
node/lucky-luke-control-plane   Ready    control-plane   5h44m   v1.30.0
node/lucky-luke-worker          Ready    <none>          5h43m   v1.30.0
node/lucky-luke-worker2         Ready    <none>          5h43m   v1.30.0

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   0/3     3            0           3s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   3         3         0       3s

NAME                                READY   STATUS              RESTARTS   AGE
pod/nginx-deploy-6bfd44d944-24lf8   0/1     ContainerCreating   0          3s
pod/nginx-deploy-6bfd44d944-5bfm4   0/1     ContainerCreating   0          3s
pod/nginx-deploy-6bfd44d944-hfgbq   0/1     ContainerCreating   0          3s
root@localhost:~#
root@localhost:~# kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-6bfd44d944-24lf8   1/1     Running   0          2m45s
pod/nginx-deploy-6bfd44d944-5bfm4   1/1     Running   0          2m45s
pod/nginx-deploy-6bfd44d944-hfgbq   1/1     Running   0          2m45s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5h47m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     3            3           2m45s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   3         3         3       2m45s

```

Changing the version of our app
```console
root@localhost:~# kubectl set image deployment/nginx-deploy nginx-container=nginx:1.9.1
deployment.apps/nginx-deploy image updated
root@localhost:~# kubectl get all
NAME                                READY   STATUS              RESTARTS   AGE
pod/nginx-deploy-6bfd44d944-24lf8   1/1     Running             0          6m37s
pod/nginx-deploy-6bfd44d944-5bfm4   1/1     Running             0          6m37s
pod/nginx-deploy-6bfd44d944-hfgbq   1/1     Running             0          6m37s
pod/nginx-deploy-c4b878cbf-s9skb    0/1     ContainerCreating   0          11s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5h51m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     1            3           6m37s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   3         3         3       6m37s
replicaset.apps/nginx-deploy-c4b878cbf    1         1         0       12s
root@localhost:~# kubectl get all
NAME                                READY   STATUS              RESTARTS   AGE
pod/nginx-deploy-6bfd44d944-5bfm4   1/1     Running             0          6m49s
pod/nginx-deploy-6bfd44d944-hfgbq   0/1     Terminating         0          6m49s
pod/nginx-deploy-c4b878cbf-2b7l2    1/1     Running             0          11s
pod/nginx-deploy-c4b878cbf-qkdql    0/1     ContainerCreating   0          1s
pod/nginx-deploy-c4b878cbf-s9skb    1/1     Running             0          23s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5h51m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     3            3           6m50s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   1         1         1       6m50s
replicaset.apps/nginx-deploy-c4b878cbf    3         3         2       25s
root@localhost:~# kubectl get all
NAME                               READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-c4b878cbf-2b7l2   1/1     Running   0          24s
pod/nginx-deploy-c4b878cbf-qkdql   1/1     Running   0          14s
pod/nginx-deploy-c4b878cbf-s9skb   1/1     Running   0          36s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5h51m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     3            3           7m2s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   0         0         0       7m2s
replicaset.apps/nginx-deploy-c4b878cbf    3         3         3       37s

```

Having the logs of our change
```console
root@localhost:~# kubectl describe deploy nginx-deploy
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Thu, 27 Jun 2024 21:01:48 +0000
Labels:                 env=demo
Annotations:            deployment.kubernetes.io/revision: 2
Selector:               env=demo-pod
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  env=demo-pod
           type=frontend
  Containers:
   nginx-container:
    Image:         nginx:1.9.1
    Port:          80/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  nginx-deploy-6bfd44d944 (0/0 replicas created)
NewReplicaSet:   nginx-deploy-c4b878cbf (3/3 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  8m43s  deployment-controller  Scaled up replica set nginx-deploy-6bfd44d944 to 3
  Normal  ScalingReplicaSet  2m17s  deployment-controller  Scaled up replica set nginx-deploy-c4b878cbf to 1
  Normal  ScalingReplicaSet  2m5s   deployment-controller  Scaled down replica set nginx-deploy-6bfd44d944 to 2 from 3
  Normal  ScalingReplicaSet  2m5s   deployment-controller  Scaled up replica set nginx-deploy-c4b878cbf to 2 from 1
  Normal  ScalingReplicaSet  115s   deployment-controller  Scaled down replica set nginx-deploy-6bfd44d944 to 1 from 2
  Normal  ScalingReplicaSet  115s   deployment-controller  Scaled up replica set nginx-deploy-c4b878cbf to 3 from 2
  Normal  ScalingReplicaSet  111s   deployment-controller  Scaled down replica set nginx-deploy-6bfd44d944 to 0 from 1
```

There are two revisions in the history of `rollout` of our deployment:
```console
root@localhost:~# kubectl rollout history deploy/nginx-deploy
deployment.apps/nginx-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

```

We can `rollback` the changes:
```console
root@localhost:~# kubectl rollout undo deploy/nginx-deploy
deployment.apps/nginx-deploy rolled back
root@localhost:~# kubectl rollout history deploy/nginx-deploy
deployment.apps/nginx-deploy
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

root@localhost:~# kubectl get all
NAME                                READY   STATUS        RESTARTS   AGE
pod/nginx-deploy-6bfd44d944-82pwb   1/1     Running       0          2s
pod/nginx-deploy-6bfd44d944-pfgrk   1/1     Running       0          5s
pod/nginx-deploy-6bfd44d944-xbpzt   1/1     Running       0          8s
pod/nginx-deploy-c4b878cbf-2b7l2    1/1     Terminating   0          5m58s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5h57m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     3            3           12m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   3         3         3       12m
replicaset.apps/nginx-deploy-c4b878cbf    0         0         0       6m11s
root@localhost:~# kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-6bfd44d944-82pwb   1/1     Running   0          14s
pod/nginx-deploy-6bfd44d944-pfgrk   1/1     Running   0          17s
pod/nginx-deploy-6bfd44d944-xbpzt   1/1     Running   0          20s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5h57m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     3            3           12m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   3         3         3       12m
replicaset.apps/nginx-deploy-c4b878cbf    0         0         0       6m23s
```

```console
root@localhost:~# kubectl describe deploy nginx-deploy
Name:                   nginx-deploy
Namespace:              default
CreationTimestamp:      Thu, 27 Jun 2024 21:01:48 +0000
Labels:                 env=demo
Annotations:            deployment.kubernetes.io/revision: 3
Selector:               env=demo-pod
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  env=demo-pod
           type=frontend
  Containers:
   nginx-container:
    Image:         nginx
    Port:          80/TCP
    Host Port:     0/TCP
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  nginx-deploy-c4b878cbf (0/0 replicas created)
NewReplicaSet:   nginx-deploy-6bfd44d944 (3/3 replicas created)
Events:
  Type    Reason             Age                From                   Message
  ----    ------             ----               ----                   -------
  Normal  ScalingReplicaSet  13m                deployment-controller  Scaled up replica set nginx-deploy-6bfd44d944 to 3
  Normal  ScalingReplicaSet  7m26s              deployment-controller  Scaled up replica set nginx-deploy-c4b878cbf to 1
  Normal  ScalingReplicaSet  7m14s              deployment-controller  Scaled down replica set nginx-deploy-6bfd44d944 to 2 from 3
  Normal  ScalingReplicaSet  7m14s              deployment-controller  Scaled up replica set nginx-deploy-c4b878cbf to 2 from 1
  Normal  ScalingReplicaSet  7m4s               deployment-controller  Scaled down replica set nginx-deploy-6bfd44d944 to 1 from 2
  Normal  ScalingReplicaSet  7m4s               deployment-controller  Scaled up replica set nginx-deploy-c4b878cbf to 3 from 2
  Normal  ScalingReplicaSet  7m                 deployment-controller  Scaled down replica set nginx-deploy-6bfd44d944 to 0 from 1
  Normal  ScalingReplicaSet  84s                deployment-controller  Scaled up replica set nginx-deploy-6bfd44d944 to 1 from 0
  Normal  ScalingReplicaSet  81s                deployment-controller  Scaled down replica set nginx-deploy-c4b878cbf to 2 from 3
  Normal  ScalingReplicaSet  76s (x4 over 81s)  deployment-controller  (combined from similar events): Scaled down replica set nginx-deploy-c4b878cbf to 0 from 1
```

```console
root@localhost:~# kubectl create deploy deploy/nginx-new --image=nginx --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: deploy/nginx-new
  name: deploy/nginx-new
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploy/nginx-new
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deploy/nginx-new
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```



