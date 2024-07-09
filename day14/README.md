## Day 14/40
# Taints and Tolerations in Kubernetes
[Video Link](https://www.youtube.com/watch?v=nwoS2tK2s6Q)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


We're going to look at `taint` and `toleration`. While a `node` has a `label` it means it has `taint` for scheduling a `workload` with that specific `label and doesn't `tolerate` other workloads to be scheduling on itself.

We taint `node` and tell a `pod` to tolerate that `taint` to be scheduled on that `node`.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0ck9ietgj62m4ffr6dkf.png)
(Photo from the video)


"Tolerations are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints. Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.

Taints and tolerations work together to ensure that pods are not scheduled onto inappropriate nodes. One or more taints are applied to a node; this marks that the node should not accept any pods that do not tolerate the taints."
**Note** There are two special cases:
- An empty `key` with operator `Exists` matches all keys, values and effects which means this will tolerate everything.

- An empty `effect` matches all effects with key `key1`.

The allowed values for the `effect` field are:
- NoExecute         > for newer and existing pods
- NoSchedule        > for newer pods
- PreferNoSchedule  > No Guaranteed

[source](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

A toleration is essentially the counter to a taint, allowing a pod to "ignore" taints applied to a node. A toleration is defined in the `pod` specification and must match the `key`, `value`, and `effect` of the `taint` it intends to `tolerate`.
Toleration Operators: While matching taints, tolerations can use operators like `Equal` and `Exists`. 
The `Equal` operator requires an exact match of `key`, `value`, and `effect`, whereas the `Exists` operator matches a taint based on the key alone, disregarding the value.
For instance:
```
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```
[source](https://overcast.blog/mastering-kubernetes-taints-and-tolerations-08756d5faf55)

---

#### 1. Taint the node
```console
root@localhost:~# kubectl get nodes
NAME                       STATUS   ROLES           AGE   VERSION
lucky-luke-control-plane   Ready    control-plane   8d    v1.30.0
lucky-luke-worker          Ready    <none>          8d    v1.30.0
lucky-luke-worker2         Ready    <none>          8d    v1.30.0
root@localhost:~# kubectl taint node lucky-luke-worker gpu=true:NoSchedule
node/lucky-luke-worker tainted
root@localhost:~# kubectl taint node lucky-luke-worker2 gpu=true:NoSchedule
node/lucky-luke-worker2 tainted
root@localhost:~# kubectl describe node lucky-luke-worker | grep -i taints
Taints:             gpu=true:NoSchedule

```

- Let's schedule a `pod`
```console
root@localhost:~# kubectl run nginx --image=nginx
pod/nginx created
root@localhost:~# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          6s

```
It says it's in `Pending` status, so let's see the error message of the pod:
```console
root@localhost:~# kubectl describe pod nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           run=nginx
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Containers:
  nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4xh8p (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-4xh8p:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  89s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 2 node(s) had untolerated taint {gpu: true}. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

```
And the message is clear to us :)
```
0/3 nodes are available.
1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }
2 node(s) had untolerated taint {gpu: true}
0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

- We need create toleration on a `pod` to be scheduled
```console
root@localhost:~# kubectl run redis --image=redis --dry-run=client -o yaml > redis_day14.yaml
root@localhost:~# vim redis_day14.yaml

```

Adding `tolerations` to `yaml` file:
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
status: {}
```

Apply the file:
```console
root@localhost:~# kubectl apply -f redis_day14.yaml
pod/redis created
root@localhost:~# kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   0/1     Pending   0          10m
redis   1/1     Running   0          5s
root@localhost:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
nginx   0/1     Pending   0          10m   <none>        <none>               <none>           <none>
redis   1/1     Running   0          17s   10.244.2.12   lucky-luke-worker2   <none>           <none>

```

Let's delete the taint of one node and see what will happen to our pending `pod`:
```console
root@localhost:~# kubectl taint node lucky-luke-worker gpu=true:NoSchedule-
node/lucky-luke-worker untainted
root@localhost:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          22m   10.244.1.14   lucky-luke-worker    <none>           <none>
redis   1/1     Running   0          11m   10.244.2.12   lucky-luke-worker2   <none>           <none>

```

- By default, the `control-plane` node has taint `NoSchedule`
```console
root@localhost:~# kubectl get nodes
NAME                       STATUS   ROLES           AGE   VERSION
lucky-luke-control-plane   Ready    control-plane   8d    v1.30.0
lucky-luke-worker          Ready    <none>          8d    v1.30.0
lucky-luke-worker2         Ready    <none>          8d    v1.30.0
root@localhost:~# kubectl describe node lucky-luke-control-plane | grep Taint
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

---

#### Selector
Instead a `node` can decide which type `pod` to accept, it will give the decision to a `pod` to which `node` can deployed on.

- Let's try:
```console
root@localhost:~# kubectl run nginx2 --image=nginx --dry-run=client -o yaml > nginx2-day14.yaml
root@localhost:~# vim nginx2-day14.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx2
  name: nginx2
spec:
  containers:
  - image: nginx
    name: nginx2
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  nodeSelector:
    gpu: "false"
status: {}
```

```console
root@localhost:~# kubectl apply -f nginx2-day14.yaml
pod/nginx2 created
root@localhost:~# kubectl get pods -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP            NODE                 NOMINATED NODE   READINESS GATES
nginx    1/1     Running   0          46m   10.244.1.14   lucky-luke-worker    <none>           <none>
nginx2   0/1     Pending   0          8s    <none>        <none>               <none>           <none>
redis    1/1     Running   0          35m   10.244.2.12   lucky-luke-worker2   <none>           <none>

```

- Label one node and let's see what will happen:
```console
root@localhost:~# kubectl label node lucky-luke-worker gpu="false"
node/lucky-luke-worker labeled
root@localhost:~# kubectl get pods -o wide
NAME     READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
nginx    1/1     Running   0          49m     10.244.1.14   lucky-luke-worker    <none>           <none>
nginx2   1/1     Running   0          3m21s   10.244.1.15   lucky-luke-worker    <none>           <none>
redis    1/1     Running   0          38m     10.244.2.12   lucky-luke-worker2   <none>           <none>
```








