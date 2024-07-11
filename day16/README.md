## Day 16/40
# Kubernetes Requests and Limits
[Video Link](https://www.youtube.com/watch?v=Q-mk6EZVX_Q)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section we're looking to `resource`, `request` and `limit`
 which is another concept for scheduling our `pod`.
With the `request` we define lower-band and with the `limit` the upper-band is defined for resources which is a pod needed
For example: [source](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#example-1)
```yaml
...
  - name: log-aggregator
    image: images.my-company.example/log-aggregator:v6
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
...
```


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kt0iy2b2q0qs8jgo9ote.png)

(Photo from the video)

- Insufficient Resources
- OOM

---

#### Demo

- Run the metrics server with the yaml file [here](https://raw.githubusercontent.com/piyushsachdeva/CKA-2024/main/Resources/Day16/metrics-server.yaml)
```console
root@localhost:~# kubectl get pods -n kube-system
NAME                                               READY   STATUS    RESTARTS     AGE
coredns-7db6d8ff4d-bftnd                           1/1     Running   1 (9d ago)   10d
coredns-7db6d8ff4d-zs54d                           1/1     Running   1 (9d ago)   10d
etcd-lucky-luke-control-plane                      1/1     Running   1 (9d ago)   10d
kindnet-fbwgj                                      1/1     Running   1 (9d ago)   10d
kindnet-hxb7v                                      1/1     Running   1 (9d ago)   10d
kindnet-kh5s6                                      1/1     Running   1 (9d ago)   10d
kube-apiserver-lucky-luke-control-plane            1/1     Running   1 (9d ago)   10d
kube-controller-manager-lucky-luke-control-plane   1/1     Running   1 (9d ago)   10d
kube-proxy-42h2f                                   1/1     Running   1 (9d ago)   10d
kube-proxy-dhzrs                                   1/1     Running   1 (9d ago)   10d
kube-proxy-rlzwk                                   1/1     Running   1 (9d ago)   10d
kube-scheduler-lucky-luke-control-plane            1/1     Running   1 (9d ago)   10d
metrics-server-55677cdb4c-c826z                    1/1     Running   0            9m54s
```

- Let's see how much resources the nodes and pods are consume
```console
root@localhost:~# kubectl top nodes
NAME                       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
lucky-luke-control-plane   80m          4%     601Mi           15%
lucky-luke-worker          19m          0%     213Mi           5%
lucky-luke-worker2         18m          0%     139Mi           3%
root@localhost:~# kubectl top pods -n kube-system
NAME                                               CPU(cores)   MEMORY(bytes)
coredns-7db6d8ff4d-bftnd                           1m           15Mi
coredns-7db6d8ff4d-zs54d                           1m           16Mi
etcd-lucky-luke-control-plane                      13m          45Mi
kindnet-fbwgj                                      1m           14Mi
kindnet-hxb7v                                      1m           13Mi
kindnet-kh5s6                                      1m           13Mi
kube-apiserver-lucky-luke-control-plane            37m          224Mi
kube-controller-manager-lucky-luke-control-plane   10m          54Mi
kube-proxy-42h2f                                   1m           17Mi
kube-proxy-dhzrs                                   1m           18Mi
kube-proxy-rlzwk                                   1m           17Mi
kube-scheduler-lucky-luke-control-plane            2m           24Mi
metrics-server-55677cdb4c-c826z                    3m           19Mi

```

We can make some stress test for our cluster in new namespace
```console
root@localhost:~# kubectl create ns mem-example
namespace/mem-example created

```

- Sample 1 for exceeding available memory 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```
- Sample 2
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-2
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-2-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
```
- Apply the 2 yaml files and see the result:
```console
root@localhost:~# kubectl  get pod -n mem-example
NAME            READY   STATUS      RESTARTS      AGE
memory-demo     1/1     Running     0             4m25s
memory-demo-2   0/1     OOMKilled   3 (31s ago)   52s

```
We can see the error `OOMKilled` for the second pod because it exceeds the memory limit.

- Sample 3
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-3
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-3-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "1000Gi"
      limits:
        memory: "1000Gi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
```

- Let's see the status of our pods:
```console
root@localhost:~# kubectl create -f mem3-request.yaml
pod/memory-demo-3 created
root@localhost:~# kubectl get pod -n mem-example
NAME            READY   STATUS             RESTARTS     AGE
memory-demo     1/1     Running            0            17m
memory-demo-2   0/1     CrashLoopBackOff   7 (3m ago)   14m
memory-demo-3   0/1     Pending            0            20s

```

- Events of the pod in `Pending` status is `Insufficient memory`:
```
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  86s   default-scheduler  0/3 nodes are available: 1 Insufficient memory, 1 node(s) had untolerated taint {gpu: true}, 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/3 nodes are available: 1 No preemption victims found for incoming pod, 2 Preemption is not helpful for scheduling.

```
