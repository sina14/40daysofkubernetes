## Day 13/40
# Static Pods, Manual Scheduling, Labels, and Selectors in Kubernetes
[Video Link](https://www.youtube.com/watch?v=6eGf7_VSbrQ)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this part, `node` selector, `label` and `selector`, static `pod` and manual `scheduling` will be covered.

There's one component named `scheduler` decides which `workload` run on which `node`.
```console
root@localhost:~# kubectl get po -n kube-system | grep scheduler
kube-scheduler-lucky-luke-control-plane            1/1     Running             1 (6d10h ago)   7d
```

---

In `Kubernetes`, a `static pod` is a concept wherein you can deploy a `pod` that is not managed by the `API-server`.

`Static pods` are directly managed by the `Kubelet` component. The `Kubelet` service is deployed with the configuration path where we can add the pod manifest for the `Kubelet` to deploy.[source](https://devopscube.com/create-static-pod-kubernetes/), [read more](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

**Note** we provisioned our `cluster` with `kind` and it's actually `kubernetes` in `docker` which means every `node` is a `container` so we get help with `docker exec` command.
```console
root@localhost:~# docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED        STATUS      PORTS                                                                                          NAMES
c0266722d131   kindest/node:v1.30.0   "/usr/local/bin/entr…"   7 days ago     Up 6 days                                                                                                  lucky-luke-worker
f791fa85c269   kindest/node:v1.30.0   "/usr/local/bin/entr…"   7 days ago     Up 6 days   0.0.0.0:30001->30001/tcp, 127.0.0.1:39283->6443/tcp                                            lucky-luke-control-plane
9c2d43b4f977   kindest/node:v1.30.0   "/usr/local/bin/entr…"   7 days ago     Up 6 days                                                                                                  lucky-luke-worker2
c9d85c72c573   weejewel/wg-easy       "docker-entrypoint.s…"   5 months ago   Up 6 days   0.0.0.0:51820->51820/udp, :::51820->51820/udp, 0.0.0.0:51821->51821/tcp, :::51821->51821/tcp   wg-easy
```
---

#### 1. Static pods and manual scheduling:

- Run bash on control-plane node
```console
root@localhost:~# docker exec -it lucky-luke-control-plane bash
root@lucky-luke-control-plane:/# pwd
/
root@lucky-luke-control-plane:/# ps ef | grep kubelet
  94749 pts/1    S+     0:00  \_ grep kubelet HOSTNAME=lucky-luke-control-plane PWD=/ container=docker HOME=/root TERM=xterm NO_PROXY= SHLVL=1 HTTPS_PROXY= HTTP_PROXY= KUBECONFIG=/etc/kubernetes/admin.conf PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin _=/usr/bin/grep

```

- Directory of manifests
`kubelet` is monitoring this directory and as soon as you remove something in the directory, the `container` will be removed from `cluster`
```console
root@lucky-luke-control-plane:/# cd /etc/kubernetes/manifests/
root@lucky-luke-control-plane:/etc/kubernetes/manifests# ls -lh
total 16K
-rw------- 1 root root 2.4K Jul  1 16:16 etcd.yaml
-rw------- 1 root root 3.9K Jul  1 16:16 kube-apiserver.yaml
-rw------- 1 root root 3.4K Jul  1 16:16 kube-controller-manager.yaml
-rw------- 1 root root 1.5K Jul  1 16:16 kube-scheduler.yaml

```

- Manual scheduling
The `scheduler` looks for `nodeName` if it's not specified in the manifest of a `workload` or `yaml` file to schedule and provision on a `node`. And if it's specified, it's not its responsible to scheduling it.

```console
root@lucky-luke-control-plane:~# kubectl run nginx --image=nginx -o yaml > nginx-pod.yaml

```

Define `nodeName`
```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-07-08T16:55:26Z"
  labels:
    run: nginx
  name: nginx
  namespace: default
  resourceVersion: "915219"
  uid: 17ebe9ec-78db-4886-95ba-3882dd141e5f
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
  nodeName: lucky-luke-worker


```
First, we need to delete the `pod` which is created
```console
root@localhost:~# kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
nginx            1/1     Running   0          9m45s
nginx-ds-946m4   1/1     Running   0          4d
nginx-ds-rslrm   1/1     Running   0          4d
root@localhost:~# kubectl delete pods/nginx
pod "nginx" deleted
```

Then runing the `pod`:
```console
root@lucky-luke-control-plane:~# kubectl apply -f nginx-pod.yaml
pod/nginx created
root@lucky-luke-control-plane:~# kubectl get pod -o wide
NAME             READY   STATUS    RESTARTS   AGE    IP            NODE                 NOMINATED NODE   READINESS GATES
nginx            1/1     Running   0          70s    10.244.1.13   lucky-luke-worker    <none>           <none>
nginx-ds-946m4   1/1     Running   0          4d     10.244.2.10   lucky-luke-worker2   <none>           <none>
nginx-ds-rslrm   1/1     Running   0          4d1h   10.244.1.12   lucky-luke-worker    <none>           <none>

```
---

#### 2. Labels and Selectors

The labels are metadata which help to filtering the resources. We also have labels in `spec` for `pods` and `selector` section that is matching the labels of pods with deployments.

```console
root@lucky-luke-control-plane:~# kubectl get pods --show-labels
NAME             READY   STATUS    RESTARTS   AGE    LABELS
nginx            1/1     Running   0          11m    run=nginx
nginx-ds-946m4   1/1     Running   0          4d1h   controller-revision-hash=76c9ffb96,env=demo,pod-template-generation=1
nginx-ds-rslrm   1/1     Running   0          4d1h   controller-revision-hash=76c9ffb96,env=demo,pod-template-generation=1
root@lucky-luke-control-plane:~# kubectl get pods --selector run=nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          11m
```

Also, we can see an additional metadata `annotations` in a `workload`, which is similar to labels and storing additional details and information related to that object.
```console
root@localhost:~# kubectl edit pod nginx
```

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"creationTimestamp":"2024-07-08T16:55:26Z","labels":{"run":"nginx"},"name":"nginx","namespace":"default","resourceVersion":"915219","uid":"17ebe9ec-78db-4886-95ba-3882dd141e5f"},"spec":{"containers":[{"image":"nginx","imagePullPolicy":"Always","name":"nginx"}],"nodeName":"lucky-luke-worker"}}
  creationTimestamp: "2024-07-08T17:09:50Z"
  labels:
    run: nginx
...
```



