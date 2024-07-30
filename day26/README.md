## Day 26/40
# Kubernetes Network Policies Explained
[Video Link](https://www.youtube.com/watch?v=eVtnevr3Rao)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


And finally, `Network Policy`!
In this section we will discuss about `network policy` in `kubernetes` cluster. It's the last topic in `Security` section.

Let's quickly have a look at to the `network flow`.
- Sample 1 a simple one:
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ohar0230tzrm21jbiqr2.png)

(Photo from the video)

- Sample 2 in a `Kubernetes` cluster:
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wmpd5jw8b9qjhgroxtpd.png)

(Photo from the video)

We need `Network Policy` because it implements certain rules, certain restrictions and certain policies in our network, so each pod can or cannot communicate with other pod based on these policies which are enforce by the `CNI` plugins.
But not every `CNI` supports `Network Policy` for instance `flannel` and `kindnet`, as we are using `kind` for running our `kubernetes` cluster, they don't support these things :).
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/42i5z5go6ftb41ga581m.png)


>If you want to control traffic flow at the IP address or port level for `TCP`, `UDP`, and `SCTP` protocols, then you might consider using Kubernetes `NetworkPolicies` for particular applications in your cluster. 
>`NetworkPolicies` are an `application-centric` construct which allow you to specify how a `pod` is allowed to communicate with various network "entities" (we use the word "entity" here to avoid overloading the more common terms such as "endpoints" and "services", which have specific `Kubernetes` connotations) over the network. 
>`NetworkPolicies` apply to a connection with a `pod` on one or both ends, and are not relevant to other connections.
[source](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

`CNI` is `deamon-set`:
```console
root@localhost:~# kubectl get ds -A
NAMESPACE     NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kindnet      3         3         3       3            3           kubernetes.io/os=linux   28d
kube-system   kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   28d
```

We're going to re-install `kind` cluster without default `CNI` so we will install our own `CNI`.
Some changes we have in our yaml file as you can see we add `extraPortMappings` for `service` availability in `kind` cluster, and `disableDefaultCNI` for installing our own `CNI`.

```yaml
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
    - containerPort: 30001
      hostPort: 30001
- role: worker
- role: worker
networking:
  disableDefaultCNI: true
  #podSubnet: 192.168.0.0/16

```

---

Installing the new cluster with `kind`

```console
root@localhost:~# kind delete clusters lucky-luke
Deleted nodes: ["lucky-luke-worker" "lucky-luke-control-plane" "lucky-luke-worker2"]
Deleted clusters: ["lucky-luke"]
root@localhost:~# kind get clusters
No kind clusters found.
root@localhost:~# kind create cluster --name=jolly-jumper --config=kind-jolly-jumper.yaml
Creating cluster "jolly-jumper" ...
 ✓ Ensuring node image (kindest/node:v1.30.0) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-jolly-jumper"
You can now use your cluster with:

kubectl cluster-info --context kind-jolly-jumper

Have a nice day! 👋
root@localhost:~# kubectl get nodes
NAME                         STATUS     ROLES           AGE   VERSION
jolly-jumper-control-plane   NotReady   control-plane   56s   v1.30.0
jolly-jumper-worker          NotReady   <none>          35s   v1.30.0
jolly-jumper-worker2         NotReady   <none>          34s   v1.30.0

```
It won't start because we don't have any `CNI`, so we are going to install a `CNI` plugin.

>Make sure you've configured a network provider with network policy support. There are a number of network providers that support NetworkPolicy, including:
>- Antrea
>- Calico
>- Cilium
>- Kube-router
>- Romana
>- Weave Net

[source](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

Let's choose the last one, [Weave Net](https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/).
Installing with following [this](https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/#install-the-weave-net-addon) instruction and [here](https://github.com/weaveworks/weave/blob/master/site/kubernetes/kube-addon.md#-installation).

---

```console
root@localhost:~# kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```

After installation of `CNI`, let's check the cluster state:
```console
root@localhost:~# kubectl get nodes
NAME                         STATUS   ROLES           AGE   VERSION
jolly-jumper-control-plane   Ready    control-plane   15m   v1.30.0
jolly-jumper-worker          Ready    <none>          15m   v1.30.0
jolly-jumper-worker2         Ready    <none>          15m   v1.30.0
root@localhost:~# kubectl get ds -A
NAMESPACE     NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-proxy   3         3         3       3            3           kubernetes.io/os=linux   15m
kube-system   weave-net    3         3         3       3            3           <none>                   99s
root@localhost:~# kubectl get pods -A
NAMESPACE            NAME                                                 READY   STATUS    RESTARTS      AGE
kube-system          coredns-7db6d8ff4d-tdkbc                             1/1     Running   0             15m
kube-system          coredns-7db6d8ff4d-tqpz9                             1/1     Running   0             15m
kube-system          etcd-jolly-jumper-control-plane                      1/1     Running   0             15m
kube-system          kube-apiserver-jolly-jumper-control-plane            1/1     Running   0             15m
kube-system          kube-controller-manager-jolly-jumper-control-plane   1/1     Running   0             15m
kube-system          kube-proxy-7s2tz                                     1/1     Running   0             15m
kube-system          kube-proxy-8bg22                                     1/1     Running   0             15m
kube-system          kube-proxy-s7m5s                                     1/1     Running   0             15m
kube-system          kube-scheduler-jolly-jumper-control-plane            1/1     Running   0             15m
kube-system          weave-net-2cnqf                                      2/2     Running   1 (94s ago)   108s
kube-system          weave-net-lx2g4                                      2/2     Running   1 (93s ago)   108s
kube-system          weave-net-z8q2q                                      2/2     Running   1 (88s ago)   108s
local-path-storage   local-path-provisioner-988d74bc-56h58                1/1     Running   0             15m
```

Our `CNI` is a `deamon-set`, also has a side-car container and both are up & running.
Now, we are going to implement a sample scenario.


#### Demo
The multiple resource in one manifest seprated by `---`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    role: frontend
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    role: frontend
spec:
  selector:
    role: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    role: backend
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - name: http
      containerPort: 80
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    role: backend
spec:
  selector:
    role: backend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: db
  labels:
    name: mysql
spec:
  selector:
    name: mysql
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306
---
apiVersion: v1
kind: Pod
metadata:
  name: mysql
  labels:
    name: mysql
spec:
  containers:
    - name: mysql
      image: mysql:latest
      env:
        - name: "MYSQL_USER"
          value: "mysql"
        - name: "MYSQL_PASSWORD"
          value: "mysql"
        - name: "MYSQL_DATABASE"
          value: "testdb"
        - name: "MYSQL_ROOT_PASSWORD"
          value: "verysecure"
      ports:
        - name: http
          containerPort: 3306
          protocol: TCP
```
[source](https://github.com/piyushsachdeva/CKA-2024/tree/main/Resources/Day26#application-manifest)

```console
root@localhost:~# kubectl apply -f day26-manifest.yaml
pod/frontend created
service/frontend created
pod/backend created
service/backend created
service/db created
pod/mysql created
root@localhost:~# kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
backend    1/1     Running   0          38s
frontend   1/1     Running   0          38s
mysql      1/1     Running   0          37s
root@localhost:~# kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
backend      ClusterIP   10.96.37.184    <none>        80/TCP     69s
db           ClusterIP   10.96.212.199   <none>        3306/TCP   69s
frontend     ClusterIP   10.96.94.82     <none>        80/TCP     69s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    33m

```

---

- Test communication from `frontend` to `backend` and `db`

```console
root@localhost:~# kubectl exec -it frontend -- bash
root@frontend:/# curl -I backend:80
HTTP/1.1 200 OK
Server: nginx/1.27.0
Date: Tue, 30 Jul 2024 17:35:16 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 28 May 2024 13:22:30 GMT
Connection: keep-alive
ETag: "6655da96-267"
Accept-Ranges: bytes

root@frontend:/# apt update && apt install telnet -y
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Get:2 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
Get:3 http://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Get:4 http://deb.debian.org/debian bookworm/main amd64 Packages [8788 kB]
Get:5 http://deb.debian.org/debian bookworm-updates/main amd64 Packages [13.8 kB]
Get:6 http://deb.debian.org/debian-security bookworm-security/main amd64 Packages [169 kB]
Fetched 9225 kB in 2s (5693 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  inetutils-telnet netbase
The following NEW packages will be installed:
  inetutils-telnet netbase telnet
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 174 kB of archives.
After this operation, 353 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bookworm/main amd64 netbase all 6.4 [12.8 kB]
Get:2 http://deb.debian.org/debian bookworm/main amd64 inetutils-telnet amd64 2:2.4-2+deb12u1 [120 kB]
Get:3 http://deb.debian.org/debian bookworm/main amd64 telnet all 0.17+2.4-2+deb12u1 [41.1 kB]
Fetched 174 kB in 0s (3124 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package netbase.
(Reading database ... 7581 files and directories currently installed.)
Preparing to unpack .../archives/netbase_6.4_all.deb ...
Unpacking netbase (6.4) ...
Selecting previously unselected package inetutils-telnet.
Preparing to unpack .../inetutils-telnet_2%3a2.4-2+deb12u1_amd64.deb ...
Unpacking inetutils-telnet (2:2.4-2+deb12u1) ...
Selecting previously unselected package telnet.
Preparing to unpack .../telnet_0.17+2.4-2+deb12u1_all.deb ...
Unpacking telnet (0.17+2.4-2+deb12u1) ...
Setting up netbase (6.4) ...
Setting up inetutils-telnet (2:2.4-2+deb12u1) ...
update-alternatives: using /usr/bin/inetutils-telnet to provide /usr/bin/telnet (telnet) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/telnet.1.gz because associated file /usr/share/man/man1/inetutils-telnet.1.gz (of link group telnet) doesn't exist
Setting up telnet (0.17+2.4-2+deb12u1) ...
root@frontend:/# telnet db 3306
Trying 10.96.212.199...
Connected to db.
Escape character is '^]'.
I
9.0.1    *2x1T▒0dKGQ6:8caching_sha2_passwordxterm-256colorxterm-256color
!#08S01Got packets out of orderConnection closed by foreign host.
root@frontend:/#
exit

```

Next, we're going to create a `NetworkPolicy` object:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mysql-access
spec:
  podSelector:
    matchLabels:
      name: mysql
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: "backend"
    ports:
      - protocol: TCP
        port: 3306
```
We restricted access to `db` only from `backend` and `frontend` cannot connect to `db`

```console
root@localhost:~# kubectl get pods --show-labels
NAME       READY   STATUS    RESTARTS   AGE   LABELS
backend    1/1     Running   0          30m   role=backend
frontend   1/1     Running   0          30m   role=frontend
mysql      1/1     Running   0          30m   name=mysql
root@localhost:~# vim day26-network-policy.yaml
root@localhost:~# kubectl apply -f day26-network-policy.yaml
networkpolicy.networking.k8s.io/mysql-access created
root@localhost:~# kubectl get networkpolicy
NAME           POD-SELECTOR   AGE
mysql-access   name=mysql     15s
root@localhost:~# kubectl exec -it frontend -- bash
root@frontend:/# telnet db 3306
Trying 10.96.212.199...
telnet: Unable to connect to remote host: Connection timed out
root@frontend:/# exit
exit
command terminated with exit code 1
root@localhost:~# kubectl exec -it backend -- bash
root@backend:/# telnet db 3306
Trying 10.96.212.199...
Connected to db.
Escape character is '^]'.
I
9.0.1
\C>vJ=9▒To;0%a=D`bcaching_sha2_password

!#08S01Got packets out of orderConnection closed by foreign host.
root@backend:/# exit
exit
root@localhost:~#

```

Check the `Network Policy`

```console
root@localhost:~# kubectl get netpol
NAME           POD-SELECTOR   AGE
mysql-access   name=mysql     6m36s
root@localhost:~# kubectl describe netpol mysql-access
Name:         mysql-access
Namespace:    default
Created on:   2024-07-30 17:49:28 +0000 UTC
Labels:       <none>
Annotations:  <none>
Spec:
  PodSelector:     name=mysql
  Allowing ingress traffic:
    To Port: 3306/TCP
    From:
      PodSelector: role=backend
  Not affecting egress traffic
  Policy Types: Ingress
```

---
---

**Note** the `weavenet` and its repository has been archived by the owner on Jun 20, 2024. It is now read-only and out of date.
Last commit at time I'm writing this article is on Nov 23, 2022 by [Eneko Fernández](https://github.com/enekofb).
In the video, time 32:08, @piyushsachdeva decided to use another `CNI` named `calico`. But I had not any problem with `weavenet` plugin to continue the scenario.

#### 1. Create a new cluster with `kind`

```sh
root@localhost:~# kind create cluster --name=joe-dalton --config=kind-joe-dalton.yaml
Creating cluster "joe-dalton" ...
 ✓ Ensuring node image (kindest/node:v1.30.0) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-joe-dalton"
You can now use your cluster with:

kubectl cluster-info --context kind-joe-dalton

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/
root@localhost:~# kubectl cluster-info --context kind-joe-dalton
Kubernetes control plane is running at https://127.0.0.1:40459
CoreDNS is running at https://127.0.0.1:40459/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
root@localhost:~# kubectl get nodes
NAME                       STATUS     ROLES           AGE   VERSION
joe-dalton-control-plane   NotReady   control-plane   54s   v1.30.0
joe-dalton-worker          NotReady   <none>          29s   v1.30.0
joe-dalton-worker2         NotReady   <none>          29s   v1.30.0

root@localhost:~# kind get clusters
joe-dalton

```

#### 2. Install `calico` plugin operator and custom resource definitions
[source](https://docs.tigera.io/calico/latest/getting-started/kubernetes/kind#install-calico)

```sh
root@localhost:~# kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
namespace/tigera-operator created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpfilters.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/apiservers.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/imagesets.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/installations.operator.tigera.io created
customresourcedefinition.apiextensions.k8s.io/tigerastatuses.operator.tigera.io created
serviceaccount/tigera-operator created
clusterrole.rbac.authorization.k8s.io/tigera-operator created
clusterrolebinding.rbac.authorization.k8s.io/tigera-operator created
deployment.apps/tigera-operator created
root@localhost:~kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yamlml
installation.operator.tigera.io/default created
apiserver.operator.tigera.io/default created

```

#### 3. Check the `calico` resources and nodes

```sh
root@localhost:~# kubectl get pods -l k8s-app=calico-node -A
NAMESPACE       NAME                READY   STATUS    RESTARTS   AGE
calico-system   calico-node-9zzdq   0/1     Running   0          77s
calico-system   calico-node-gmxb2   0/1     Running   0          77s
calico-system   calico-node-n6vsg   0/1     Running   0          77s
root@localhost:~# kubectl get nodes
NAME                       STATUS   ROLES           AGE     VERSION
joe-dalton-control-plane   Ready    control-plane   6m20s   v1.30.0
joe-dalton-worker          Ready    <none>          5m55s   v1.30.0
joe-dalton-worker2         Ready    <none>          5m55s   v1.30.0

```

```sh
root@localhost:~# kubectl get pod -n calico-system
NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-57ff4cfd4f-lddpd   1/1     Running   0          3m39s
calico-node-9zzdq                          1/1     Running   0          3m39s
calico-node-gmxb2                          1/1     Running   0          3m39s
calico-node-n6vsg                          1/1     Running   0          3m39s
calico-typha-955bb95d5-9j8vq               1/1     Running   0          3m39s
calico-typha-955bb95d5-sk2vf               1/1     Running   0          3m37s
csi-node-driver-6fm85                      2/2     Running   0          3m39s
csi-node-driver-brkx6                      2/2     Running   0          3m39s
csi-node-driver-ws8ks                      2/2     Running   0          3m39s

```

#### 4. Apply the manifest

```sh
root@localhost:~# kubectl apply -f day26-manifest.yaml
pod/frontend created
service/frontend created
pod/backend created
service/backend created
service/db created
pod/mysql created
root@localhost:~# kubectl get pod,svc
NAME           READY   STATUS    RESTARTS   AGE
pod/backend    1/1     Running   0          64s
pod/frontend   1/1     Running   0          64s
pod/mysql      1/1     Running   0          64s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/backend      ClusterIP   10.96.105.154   <none>        80/TCP     64s
service/db           ClusterIP   10.96.207.211   <none>        3306/TCP   64s
service/frontend     ClusterIP   10.96.201.90    <none>        80/TCP     64s
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    9m45s
root@localhost:~# kubectl apply -f day26-network-policy.yaml
networkpolicy.networking.k8s.io/mysql-access created

```

#### 5. Check access from `frontend` pod

```sh
root@localhost:~# kubectl exec -it pod/frontend -- bash
root@frontend:/# ip a
bash: ip: command not found
root@frontend:/# apt update && apt install telnet -y
Get:1 http://deb.debian.org/debian bookworm InRelease [151 kB]
Get:2 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
Get:3 http://deb.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Get:4 http://deb.debian.org/debian bookworm/main amd64 Packages [8788 kB]
Get:5 http://deb.debian.org/debian bookworm-updates/main amd64 Packages [13.8 kB]
Get:6 http://deb.debian.org/debian-security bookworm-security/main amd64 Packages [169 kB]
Fetched 9225 kB in 2s (4543 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  inetutils-telnet netbase
The following NEW packages will be installed:
  inetutils-telnet netbase telnet
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 174 kB of archives.
After this operation, 353 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bookworm/main amd64 netbase all 6.4 [12.8 kB]
Get:2 http://deb.debian.org/debian bookworm/main amd64 inetutils-telnet amd64 2:2.4-2+deb12u1 [120 kB]
Get:3 http://deb.debian.org/debian bookworm/main amd64 telnet all 0.17+2.4-2+deb12u1 [41.1 kB]
Fetched 174 kB in 0s (2787 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package netbase.
(Reading database ... 7581 files and directories currently installed.)
Preparing to unpack .../archives/netbase_6.4_all.deb ...
Unpacking netbase (6.4) ...
Selecting previously unselected package inetutils-telnet.
Preparing to unpack .../inetutils-telnet_2%3a2.4-2+deb12u1_amd64.deb ...
Unpacking inetutils-telnet (2:2.4-2+deb12u1) ...
Selecting previously unselected package telnet.
Preparing to unpack .../telnet_0.17+2.4-2+deb12u1_all.deb ...
Unpacking telnet (0.17+2.4-2+deb12u1) ...
Setting up netbase (6.4) ...
Setting up inetutils-telnet (2:2.4-2+deb12u1) ...
update-alternatives: using /usr/bin/inetutils-telnet to provide /usr/bin/telnet (telnet) in auto mode
update-alternatives: warning: skip creation of /usr/share/man/man1/telnet.1.gz because associated file /usr/share/man/man1/inetutils-telnet.1.gz (of link group telnet) doesn't exist
Setting up telnet (0.17+2.4-2+deb12u1) ...
root@frontend:/# curl -I backend:80
HTTP/1.1 200 OK
Server: nginx/1.27.0
Date: Tue, 30 Jul 2024 19:08:55 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 28 May 2024 13:22:30 GMT
Connection: keep-alive
ETag: "6655da96-267"
Accept-Ranges: bytes

root@frontend:/# telnet db 3306
Trying 10.96.207.211...
telnet: Unable to connect to remote host: Connection timed out

```

#### 6. Check access from `backend` pod

```sh
root@localhost:~# kubectl exec -it backend -- bash
root@backend:/# telnet db 3306
Trying 10.96.207.211...
Connected to db.
Escape character is '^]'.
I
s|?t\aching_sha2_password

!#08S01Got packets out of orderConnection closed by foreign host.
```



























