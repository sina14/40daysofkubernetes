## Day 10/40
# Kubernetes Namespace Explained
[Video Link](https://www.youtube.com/watch?v=yVLXIydlU_0)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section, we're going to explain `namespace` in `kubernetes`.
It's a method by which single `cluster` used by and organization can be divided and categorized into multiple sub-clusters and managed individually. Different projects run simultaneously with different teams and departments work in parallel. [source](https://www.armosec.io/glossary/kubernetes-namespace/)

When we create a workload like `pod`, `deployment`, `service` and so on without mentioning a `namespace`, it's actually created in `default` `namespace`.

By provisioning a `kubernetes` cluster, it creates own `namespace` named `kube-system` and all of its components will be in the `kube-system` namespace.

Each workload for example `pod` in a `namespace` can easily interact with each other with the `hostname` of their `pod`. But for interact to other `namespace` `pod`, they have to use `FQDN`.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fl5lmqiqjrrx0hhistuj.png)
(Photo from the video)

- Get all `namespace`s:
```cosnole
root@localhost:~# kubectl get namespaces
NAME                 STATUS   AGE
default              Active   2d
kube-node-lease      Active   2d
kube-public          Active   2d
kube-system          Active   2d
local-path-storage   Active   2d
```
- Get all in `kube-system` namespace:
```console
root@localhost:~# kubectl get all --namespace=kube-system
NAME                                                   READY   STATUS    RESTARTS      AGE
pod/coredns-7db6d8ff4d-bftnd                           1/1     Running   1 (33h ago)   2d
pod/coredns-7db6d8ff4d-zs54d                           1/1     Running   1 (33h ago)   2d
pod/etcd-lucky-luke-control-plane                      1/1     Running   1 (33h ago)   2d
pod/kindnet-fbwgj                                      1/1     Running   1 (33h ago)   2d
pod/kindnet-hxb7v                                      1/1     Running   1 (33h ago)   2d
pod/kindnet-kh5s6                                      1/1     Running   1 (33h ago)   2d
pod/kube-apiserver-lucky-luke-control-plane            1/1     Running   1 (33h ago)   2d
pod/kube-controller-manager-lucky-luke-control-plane   1/1     Running   1 (33h ago)   2d
pod/kube-proxy-42h2f                                   1/1     Running   1 (33h ago)   2d
pod/kube-proxy-dhzrs                                   1/1     Running   1 (33h ago)   2d
pod/kube-proxy-rlzwk                                   1/1     Running   1 (33h ago)   2d
pod/kube-scheduler-lucky-luke-control-plane            1/1     Running   1 (33h ago)   2d

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   2d

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kindnet      3         3         3       3            3           kubernetes.io/os=linux   2d
daemonset.apps/kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   2d

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2/2     2            2           2d

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-7db6d8ff4d   2         2         2       2d
```
**Note** we're using `kind`!

```console
root@localhost:~# kubectl get all
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d
root@localhost:~# kubectl get all -n=default
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d
root@localhost:~# kubectl get all --namespace=default
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d
```

---

#### 1. Create/Delete `namespace`
in declarative way:
```yaml
---
apiVersion: v1

kind: Namespace

metadata:
  name: demo
```

```console
root@localhost:~# vim namespace.yaml
root@localhost:~# kubectl create -f namespace.yaml
namespace/demo created
root@localhost:~# kubectl get namespaces
NAME                 STATUS   AGE
default              Active   2d
demo                 Active   9s
kube-node-lease      Active   2d
kube-public          Active   2d
kube-system          Active   2d
local-path-storage   Active   2d

```

- Delete `namespace`
```console
root@localhost:~# kubectl delete ns/demo
namespace "demo" deleted
```

in imperative way:
```console
root@localhost:~# kubectl create ns demo
namespace/demo created
root@localhost:~# kubectl get ns
NAME                 STATUS   AGE
default              Active   2d
demo                 Active   8s
kube-node-lease      Active   2d
kube-public          Active   2d
kube-system          Active   2d
local-path-storage   Active   2d
```

#### 2. Create/Delete `deployment` in a `namespace`
```console
root@localhost:~# kubectl create deploy nginx-demo --image=nginx -n demo
deployment.apps/nginx-demo created
root@localhost:~# kubectl get deploy
No resources found in default namespace.
root@localhost:~# kubectl get deploy -n demo
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-demo   1/1     1            1           18s

```

#### 3. Communicate between 2 namespaces:
```console
root@localhost:~# kubectl get all -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE                 NOMINATED NODE   READINESS GATES
pod/nginx-test-574bc578fc-cj5fb   1/1     Running   0          5m16s   10.244.2.2   lucky-luke-worker2   <none>           <none>

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d    <none>

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES   SELECTOR
deployment.apps/nginx-test   1/1     1            1           5m16s   nginx        nginx    app=nginx-test

NAME                                    DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES   SELECTOR
replicaset.apps/nginx-test-574bc578fc   1         1         1       5m16s   nginx        nginx    app=nginx-test,pod-template-hash=574bc578fc

root@localhost:~# kubectl get all -n demo -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP           NODE                NOMINATED NODE   READINESS GATES
pod/nginx-demo-745b695d7d-hh28q   1/1     Running   0          7m48s   10.244.1.2   lucky-luke-worker   <none>           <none>

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES   SELECTOR
deployment.apps/nginx-demo   1/1     1            1           7m48s   nginx        nginx    app=nginx-demo

NAME                                    DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES   SELECTOR
replicaset.apps/nginx-demo-745b695d7d   1         1         1       7m48s   nginx        nginx    app=nginx-demo,pod-template-hash=745b695d7d

```
- Both can have communication and we test it by `curl` with `ip`

```console
# curl 10.244.2.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
# curl 10.244.1.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

- Scaling both `deployment`
```console
root@localhost:~# kubectl scale --replicas=3 deploy nginx-test
deployment.apps/nginx-test scaled
root@localhost:~# kubectl scale --replicas=3 deploy nginx-demo -n demo
deployment.apps/nginx-demo scaled

root@localhost:~# kubectl get pod,deploy
NAME                              READY   STATUS    RESTARTS   AGE
pod/nginx-test-574bc578fc-cj5fb   1/1     Running   0          14m
pod/nginx-test-574bc578fc-cqj5r   1/1     Running   0          10s
pod/nginx-test-574bc578fc-nxt4j   1/1     Running   0          10s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-test   3/3     3            3           14m
root@localhost:~# kubectl get pod,deploy -n demo
NAME                              READY   STATUS    RESTARTS   AGE
pod/nginx-demo-745b695d7d-hh28q   1/1     Running   0          16m
pod/nginx-demo-745b695d7d-rp88f   1/1     Running   0          19s
pod/nginx-demo-745b695d7d-sc8rg   1/1     Running   0          19s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-demo   3/3     3            3           16m

```

- Creating `service` for both `deployment`

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ixs4pp3qzju16y9c62sw.png)
(Photo from the video)

Test in `default` namespace:
```console
root@localhost:~# kubectl expose --name svc-test deploy/nginx-test --port 80
service/svc-test exposed
root@localhost:~# kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-test-574bc578fc-cj5fb   1/1     Running   0          29m
nginx-test-574bc578fc-cqj5r   1/1     Running   0          15m
nginx-test-574bc578fc-nxt4j   1/1     Running   0          15m
root@localhost:~# kubectl exec -it nginx-test-574bc578fc-cj5fb -- sh
# cat /etc/resolv.conf
search default.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
# curl svc-demo.demo.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```
Test in `demo` namespace:
```console
root@localhost:~# kubectl expose --name svc-demo deploy/nginx-demo --port 80 -n demo
service/svc-demo exposed
root@localhost:~# kubectl get pod -n demo
NAME                          READY   STATUS    RESTARTS   AGE
nginx-demo-745b695d7d-hh28q   1/1     Running   0          35m
nginx-demo-745b695d7d-rp88f   1/1     Running   0          19m
nginx-demo-745b695d7d-sc8rg   1/1     Running   0          19m
root@localhost:~# kubectl exec -it nginx-demo-745b695d7d-hh28q -n demo -- sh
# cat /etc/resolv.conf
search demo.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
# curl svc-test.default.svc.cluster.local
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```














