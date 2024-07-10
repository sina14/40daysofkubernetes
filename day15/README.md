## Day 15/40
# Kubernetes Node Affinity Explained
[Video Link](https://www.youtube.com/watch?v=5vimzBRnoDk)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


We're going to understand node `affinity` in `kubernetes` system.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3gr7dcuz9ygnbla6ey10.png)

(Photo from the video)

---

Node `affinity` in `Kubernetes` is a set of rules used to specify preferences that affect how pods are placed on nodes. It allows you to constrain which nodes your `pod` is eligible to be scheduled on, based on labels on nodes and if those labels match the rules.

Types of Node Affinity:
- **RequiredDuringSchedulingIgnoredDuringExecution**: Pods will only be placed on nodes that match the specified rules. If no matching nodes are available, the pods won’t be scheduled.
- **PreferredDuringSchedulingIgnoredDuringExecution**: Specifies preferences that the scheduler will attempt to enforce but will not guarantee.

[source](https://overcast.blog/mastering-node-affinity-and-anti-affinity-in-kubernetes-db769af90f5c)

In simple words:
**RequiredDuringSchedulingIgnoredDuringExecution**
- It will make sure that the `pod` only get scheduled when the operator matches with the label.

**PreferredDuringSchedulingIgnoredDuringExecution**
- It will try to matches the operator with labels, if it doesn't find, even that schedule the `pod` on any `node`.

---

#### Demo
[source](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/#schedule-a-pod-using-preferred-node-affinity)
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
  containers:
  - name: nginx
    image: nginx

```
- Run the pod:
```console
root@localhost:~# kubectl get pods --show-labels -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES   LABELS
nginx   0/1     Pending   0          8s    <none>   <none>   <none>           <none>            <none>

```

- See the logs:
```
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  82s   default-scheduler  0/3 nodes are available: 1 node(s) didn't match Pod's node affinity/selector, 1 node(s) had untolerated taint {gpu: true}, 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```

- Let's add label to a node and see what will be happened:
```console
root@localhost:~# kubectl label node lucky-luke-worker disktype=ssd
node/lucky-luke-worker labeled
root@localhost:~# kubectl get pods --show-labels -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP            NODE                NOMINATED NODE   READINESS GATES   LABELS
nginx   1/1     Running   0          6m19s   10.244.1.17   lucky-luke-worker   <none>           <none>            <none>

```

- Another example:
```yaml
---
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
            - key: disktype
              operator: In
              values:
                - hdd
  containers:
  - name: redis
    image: redis

```


















