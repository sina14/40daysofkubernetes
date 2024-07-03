## Day 11/40
# Multi Container Pod Kubernetes - Sidecar vs Init Container
[Video Link](https://www.youtube.com/watch?v=yRiFq1ykBxc)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


We're going to look at to side-car or multi `container` pods.
Let's say we have a `pod` which has a `nginx` container. It can have additional `container` inside or sidecar (helper) `nginx` container that for instance a `health-check` or `init-container`, running certain tasks, for the main `container`, in this example it's `nginx`.

The first container that is run is `init-container`. As soon as the tasks of `init-container` completed, the main `container` starts. They also share the resources in that `pod`, such as `memory` or `cpu`.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0fb38v6xiwqi14i60fps.png)
(Photo from the video)


#### 1. Create `pod`
```yaml
---
apiVersion: v1

kind: Pod

metadata:
  name: myapp
  labels:
    name: myapp-pod

spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      env:
        - name: FIRSTNAME
          value: "SINA"
```

```console
root@localhost:~# kubectl apply -f pod-sidecar.yaml
pod/myapp created
root@localhost:~# kubectl get pod
NAME    READY   STATUS      RESTARTS     AGE
myapp   0/1     Completed   1 (2s ago)   3s
root@localhost:~# kubectl get pod
NAME    READY   STATUS             RESTARTS      AGE
myapp   0/1     CrashLoopBackOff   1 (13s ago)   14s
```
**Note** Because we're not doing anything in the `pod`, the status is `CrashLoopBackOff`.

The events of the creation `pod` is:
```
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  2m13s                 default-scheduler  Successfully assigned default/myapp to lucky-luke-worker
  Normal   Pulled     44s (x5 over 2m13s)   kubelet            Container image "busybox:1.28" already present on machine
  Normal   Created    44s (x5 over 2m13s)   kubelet            Created container myapp-container
  Normal   Started    44s (x5 over 2m13s)   kubelet            Started container myapp-container
  Warning  BackOff    15s (x10 over 2m12s)  kubelet            Back-off restarting failed container myapp-container in pod myapp_default(e3371c0f-ea2a-44b6-81f7-98953931f6ad)
```

#### 2. Create multi `pod`
```yaml
---
apiVersion: v1

kind: Pod

metadata:
  name: myapp
  labels:
    name: myapp-pod

spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo the app is running && sleep 3600']
      env:
        - name: FIRSTNAME
          value: "SINA"
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command: ['sh', '-c']
      args: ['until nslookup myservice.default.svc.cluster.local; do echo Wainting for service to be up; sleep 2; done']

```
- Run the `pod`:
```console
root@localhost:~# kubectl apply -f pod-sidecar.yaml
pod/myapp created
root@localhost:~# kubectl get pod
NAME    READY   STATUS     RESTARTS   AGE
myapp   0/1     Init:0/1   0          4s
root@localhost:~# kubectl get pod
NAME    READY   STATUS     RESTARTS   AGE
myapp   0/1     Init:0/1   0          16s
root@localhost:~# kubectl get pod
NAME    READY   STATUS     RESTARTS   AGE
myapp   0/1     Init:0/1   0          42s

```
- Let's see the events of the `pod`:
```
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  2m5s  default-scheduler  Successfully assigned default/myapp to lucky-luke-worker
  Normal  Pulled     2m5s  kubelet            Container image "busybox:1.28" already present on machine
  Normal  Created    2m5s  kubelet            Created container init-myservice
  Normal  Started    2m5s  kubelet            Started container init-myservice

```
It waits until the `myservice.default.svc.local` run.

- Log of the `pod`:
```console
root@localhost:~# kubectl logs pod/myapp
Defaulted container "myapp-container" out of: myapp-container, init-myservice (init)
Error from server (BadRequest): container "myapp-container" in pod "myapp" is waiting to start: PodInitializing
```

- Logs of `init-container`:
```console
root@localhost:~# kubectl logs pod/myapp -c init-myservice
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'myservice.default.svc.local'
Wainting for service to be up
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'myservice.default.svc.local'
Wainting for service to be up
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
...
```

- Create deployment and service and see our `myapp` container is `Running`.
```console
root@localhost:~# kubectl create deploy nginx-deploy --image nginx --port 80
deployment.apps/nginx-deploy created
root@localhost:~# kubectl get pod
NAME                            READY   STATUS     RESTARTS   AGE
myapp                           0/1     Init:0/1   0          108s
nginx-deploy-7d54cf5979-qrqq4   1/1     Running    0          5s
root@localhost:~# kubectl get pod,deploy,svc
NAME                                READY   STATUS     RESTARTS   AGE
pod/myapp                           0/1     Init:0/1   0          117s
pod/nginx-deploy-7d54cf5979-qrqq4   1/1     Running    0          14s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           14s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d2h
root@localhost:~# kubectl expose deploy nginx-deploy --name myservice --port 80
service/myservice exposed
root@localhost:~# kubectl get pod,deploy,svc
root@localhost:~# kubectl get pod,deploy,svc
NAME                                READY   STATUS     RESTARTS   AGE
pod/myapp                           0/1     Init:0/1   0          2m31s
pod/nginx-deploy-7d54cf5979-qrqq4   1/1     Running    0          48s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           48s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   2d2h
service/myservice    ClusterIP   10.96.143.163   <none>        80/TCP    4s
root@localhost:~# kubectl get pod,deploy,svc
NAME                                READY   STATUS    RESTARTS   AGE
pod/myapp                           1/1     Running   0          2m42s
pod/nginx-deploy-7d54cf5979-qrqq4   1/1     Running   0          59s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           59s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   2d2h
service/myservice    ClusterIP   10.96.143.163   <none>        80/TCP    15s
root@localhost:~# kubectl logs myapp -c myapp-container
the app is running

```

- Print environment variables of the `pod`
As we can see, our variable `FIRSTNAME` is defined.
```console
root@localhost:~# kubectl exec -it myapp -- printenv
Defaulted container "myapp-container" out of: myapp-container, init-myservice (init)
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=myapp
`FIRSTNAME=SINA`
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
MYSERVICE_SERVICE_HOST=10.96.143.163
KUBERNETES_PORT=tcp://10.96.0.1:443
MYSERVICE_PORT=tcp://10.96.143.163:80
MYSERVICE_PORT_80_TCP_ADDR=10.96.143.163
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
MYSERVICE_PORT_80_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT_443_TCP_PORT=443
MYSERVICE_SERVICE_PORT=80
MYSERVICE_PORT_80_TCP=tcp://10.96.143.163:80
MYSERVICE_PORT_80_TCP_PORT=80
TERM=xterm
HOME=/root
```

#### 3. 3-Containers Pod

```yaml
---
apiVersion: v1

kind: Pod

metadata:
  name: myapp
  labels:
    name: myapp-pod

spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo the app is running && sleep 3600']
      env:
        - name: FIRSTNAME
          value: "SINA"
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command: ['sh', '-c']
      args: ['until nslookup myservice.default.svc.cluster.local; do echo Wainting for Service to be up; sleep 30; done']
    - name: init-mydb
      image: busybox:1.28
      command: ['sh', '-c']
      args: ['until nslookup mydb.default.svc.cluster.local; do echo Wainting for DB to be up; sleep 30; done']

```

```console
root@localhost:~# kubectl create deploy nginx-deploy --image nginx --port 80
deployment.apps/nginx-deploy created
root@localhost:~# kubectl expose deploy nginx-deploy --name myservice --port 80
service/myservice exposed
root@localhost:~# kubectl get pod,deploy,svc
NAME                                READY   STATUS     RESTARTS   AGE
pod/myapp                           0/1     Init:0/2   0          14m
pod/nginx-deploy-7d54cf5979-lddqf   1/1     Running    0          49s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           49s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   2d4h
service/myservice    ClusterIP   10.96.125.99   <none>        80/TCP    5s
root@localhost:~# kubectl get pod,deploy,svc
NAME                                READY   STATUS     RESTARTS   AGE
pod/myapp                           0/1     Init:0/2   0          14m
pod/nginx-deploy-7d54cf5979-lddqf   1/1     Running    0          57s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           57s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   2d4h
service/myservice    ClusterIP   10.96.125.99   <none>        80/TCP    13s
root@localhost:~# kubectl create deploy redis-deploy --image redis --port 6379
deployment.apps/redis-deploy created
root@localhost:~# kubectl expose deploy redis-deploy --name mydb --port 6379
service/mydb exposed
root@localhost:~# kubectl get pod,deploy,svc
NAME                                READY   STATUS     RESTARTS   AGE
pod/myapp                           0/1     Init:1/2   0          16m
pod/nginx-deploy-7d54cf5979-lddqf   1/1     Running    0          2m36s
pod/redis-deploy-6dd4dc84bc-2xcw9   1/1     Running    0          41s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           2m36s
deployment.apps/redis-deploy   1/1     1            1           41s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    2d4h
service/mydb         ClusterIP   10.96.147.66   <none>        6379/TCP   5s
service/myservice    ClusterIP   10.96.125.99   <none>        80/TCP     112s
root@localhost:~# kubectl get pod,deploy,svc
NAME                                READY   STATUS     RESTARTS   AGE
pod/myapp                           0/1     Init:1/2   0          16m
pod/nginx-deploy-7d54cf5979-lddqf   1/1     Running    0          2m49s
pod/redis-deploy-6dd4dc84bc-2xcw9   1/1     Running    0          54s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           2m49s
deployment.apps/redis-deploy   1/1     1            1           54s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    2d4h
service/mydb         ClusterIP   10.96.147.66   <none>        6379/TCP   18s
service/myservice    ClusterIP   10.96.125.99   <none>        80/TCP     2m5s
root@localhost:~# kubectl get pod,deploy,svc
NAME                                READY   STATUS    RESTARTS   AGE
pod/myapp                           1/1     Running   0          17m
pod/nginx-deploy-7d54cf5979-lddqf   1/1     Running   0          3m31s
pod/redis-deploy-6dd4dc84bc-2xcw9   1/1     Running   0          96s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           3m31s
deployment.apps/redis-deploy   1/1     1            1           96s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP    2d4h
service/mydb         ClusterIP   10.96.147.66   <none>        6379/TCP   60s
service/myservice    ClusterIP   10.96.125.99   <none>        80/TCP     2m47s
root@localhost:~# kubectl logs pod/myapp -c init-mydb
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Wainting for DB to be up
nslookup: can't resolve 'mydb.default.svc.cluster.local'
nslookup: can't resolve 'mydb.default.svc.cluster.local'
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Wainting for DB to be up
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Wainting for DB to be up
nslookup: can't resolve 'mydb.default.svc.cluster.local'
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Wainting for DB to be up
nslookup: can't resolve 'mydb.default.svc.cluster.local'
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      mydb.default.svc.cluster.local
Address 1: 10.96.147.66 mydb.default.svc.cluster.local
root@localhost:~# kubectl logs pod/myapp -c init-myservice
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'myservice.default.svc.cluster.local'
Wainting for Service to be up
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

nslookup: can't resolve 'myservice.default.svc.cluster.local'
Wainting for Service to be up
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

```


















