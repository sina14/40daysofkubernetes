## Day 31/40
# Understanding CoreDNS In Kubernetes
[Video Link](https://www.youtube.com/watch?v=VcWpZoRAQXE)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section, we're looking at `coredns` concept in `kubernetes`.


workloads in `kubernetes` cluster can communicate with each other with `coredns`.

```bash
root@localhost:~# kubectl get pod -n=kube-system
NAME                                               READY   STATUS    RESTARTS   AGE
coredns-7db6d8ff4d-7dwv7                           1/1     Running   0          9d
coredns-7db6d8ff4d-tmb52                           1/1     Running   0          9d
...
root@localhost:~# kubectl get svc -n=kube-system
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   9d

```

```bash
root@localhost:~# kubectl get pod
NAME          READY   STATUS    RESTARTS   AGE
task-pv-pod   1/1     Running   0          3d21h
root@localhost:~# kubectl exec task-pv-pod -- cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5

```
As we can see, the `dns` is set in `resolv.conf` file inside the `pod`, so everything across the `cluster` can be resolved with `coredns` service.

```bash
root@localhost:~# kubectl exec task-pv-pod -- cat /etc/hosts
# Kubernetes-managed hosts file.
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-mcastprefix
fe00::1 ip6-allnodes
fe00::2 ip6-allrouters
192.168.2.202   task-pv-pod
root@localhost:~# kubectl exec task-pv-pod -- hostname -i
192.168.2.202

```

Let's check something on one of the `coredns` pods:

```bash
root@localhost:~# kubectl describe pod coredns-7db6d8ff4d-7dwv7 -n=kube-system
Name:                 coredns-7db6d8ff4d-7dwv7
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Service Account:      coredns
...
    Image:         registry.k8s.io/coredns/coredns:v1.11.1
...
    Mounts:
      /etc/coredns from config-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-t6mss (ro)
...
Volumes:
  config-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      coredns
    Optional:  false
...

```

There's a `ConfigMap` for `coredns` which is mounted as `volume` to the `pod`.

```bash
root@localhost:~# kubectl get cm -n=kube-system
NAME                                                   DATA   AGE
coredns                                                1      9d
...

root@localhost:~# kubectl describe cm coredns -n=kube-system
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}


BinaryData
====

Events:  <none>

```

The official documentation is [here](https://kubernetes.io/docs/tasks/administer-cluster/coredns/) and for debugging is [here](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/) and sample `ConfigMap` yaml file:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        log
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }

```

















