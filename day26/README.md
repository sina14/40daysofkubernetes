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
```

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
- Antrea
- Calico
- Cilium
- Kube-router
- Romana
- Weave Net

[source](https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/)

Let's choose the last one, [Weave Net](https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/).
Installing with following [this](https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/#install-the-weave-net-addon) instruction and [here](https://github.com/weaveworks/weave/blob/master/site/kubernetes/kube-addon.md#-installation).
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











