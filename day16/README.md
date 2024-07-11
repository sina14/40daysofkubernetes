## Day 16/40
# Kubernetes Requests and Limits
[Video Link](https://www.youtube.com/watch?v=Q-mk6EZVX_Q)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section we're looking to `resource`, `request` and `limit`
 which is another concept for scheduling our `pod`.


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
