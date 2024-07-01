## Day 9/40
# Kubernetes Services Explained - ClusterIP vs NodePort vs Loadbalancer vs External
[Video Link](https://www.youtube.com/watch?v=tHAQWLKMTB0)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


We're going to look at `service` in `kubernetes` and its types as below list:
1. ClusterIP
2. NodePort
3. LoadBalancer
4. External Name

---

The goal is to expose our front-end app to users, we use `service` and its benefits.
As you can see in the below diagram, we are using `service` anywhere which it makes sure there is at least one `pod` serving and listening on specific `port` in our `cluster` and it's accessible only to what we specified.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k1n7ixvzbu5fmgx3gniq.png)
(Photo from the video)


### 1. Node Port
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zu4jmv64z4gygbrthmnm.png)
(Photo from the video)
These services are ideal for applications that need to be accessible from outside the `cluster`, such as web applications or APIs. With `NodePort` services, we can access our application using the node’s IP address and the port number assigned to the service.
When we create a `NodePort` service, `Kubernetes` assigns a port number from a predefined range of 30000-32767.([source](https://www.baeldung.com/ops/kubernetes-service-types#2-nodeport-services))

There are 3 ports to define:
1. `nodePort` - for external users
2. `port` - for internal clients
3. `targetPort` - container port
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0hpadf717mrxkvzya4t6.png)
(Photo from the video)
multi-pod scenario
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0cp9rkvvz16dspgke8as.png)
(Photo from the video)

#### Implementing
```console
root@localhost:~# kubectl get nodes
NAME                       STATUS   ROLES           AGE   VERSION
lucky-luke-control-plane   Ready    control-plane   4d    v1.30.0
lucky-luke-worker          Ready    <none>          4d    v1.30.0
lucky-luke-worker2         Ready    <none>          4d    v1.30.0

```
```yaml
apiVersion: v1

kind: Service

metadata:
  name: nodeport-svc
  labels:
    env: demo

spec:
  selector:
      env: demo
  type:
    NodePort
  ports:
    - nodePort: 30001
      port: 80
      targetPort: 80
```

```console
root@localhost:~# kubectl apply -f nodeport.yaml
service/nodeport-svc created
root@localhost:~# kubectl get svc
NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes     ClusterIP   10.96.0.1     <none>        443/TCP        4d
nodeport-svc   NodePort    10.96.37.68   <none>        80:30001/TCP   15s
root@localhost:~# kubectl get svc -o wide
NAME           TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE   SELECTOR
kubernetes     ClusterIP   10.96.0.1     <none>        443/TCP        4d    <none>
nodeport-svc   NodePort    10.96.37.68   <none>        80:30001/TCP   23s   env=demo
```
**Note** because we are using the `kind` for creating our `cluster`, we need to do extra step to expose our port outside the `kind`.
[Mapping ports to the host machine](https://kind.sigs.k8s.io/docs/user/quick-start/#mapping-ports-to-the-host-machine)
It needs recreating the cluster because of:
"_It's not said explicitly in the official docs, but I found some references that confirm: your thoughts are correct and changing extraPortMappings (as well as other cluster settings) is only possible with recreation of the kind cluster._"
[source](https://stackoverflow.com/a/68268591)
So our `kind` cluster would be:
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
```
- Delete the current cluster
```console
root@localhost:~# kind delete cluster --name `kind get clusters`
Deleting cluster "lucky-luke" ...
Deleted nodes: ["lucky-luke-control-plane" "lucky-luke-worker2" "lucky-luke-worker"]
```
- Create the cluster again with new feature
```console
root@localhost:~# kind create cluster --config kind-lucky-luke.yaml --name lucky-luke
Creating cluster "lucky-luke" ...
 ✓ Ensuring node image (kindest/node:v1.30.0) 🖼
 ✓ Preparing nodes 📦 📦 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
 ✓ Joining worker nodes 🚜
Set kubectl context to "kind-lucky-luke"
You can now use your cluster with:

kubectl cluster-info --context kind-lucky-luke

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
```
Agian with our `cluster`:
```console
root@localhost:~# kind get clusters
lucky-luke
root@localhost:~# kubectl get nodes
NAME                       STATUS   ROLES           AGE     VERSION
lucky-luke-control-plane   Ready    control-plane   9m14s   v1.30.0
lucky-luke-worker          Ready    <none>          8m47s   v1.30.0
lucky-luke-worker2         Ready    <none>          8m47s   v1.30.0
```

- Create deployment (in day 8/40):
```yaml
---
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-deploy
  labels:
    env: demo

spec:
  template:
    metadata:
      name: nginx-pod
      labels:
        env: demo
        type: frontend

    spec:
      containers:
      - name: nginx-container
        image: nginx
        ports:
          - containerPort: 80

  replicas: 3
  selector:
    matchLabels:
      env: demo
```

```console
root@localhost:~# kubectl create -f day08-deploy.yaml
deployment.apps/nginx-deploy created
root@localhost:~# kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-6bfd44d944-dlfcf   1/1     Running   0          71s
pod/nginx-deploy-6bfd44d944-rgmjn   1/1     Running   0          71s
pod/nginx-deploy-6bfd44d944-tpvfz   1/1     Running   0          71s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   14m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     3            3           71s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   3         3         3       71s
```

- Create `service`:
```console
root@localhost:~# kubectl apply -f nodeport.yaml
service/nodeport-svc created
root@localhost:~# kubectl get all
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-6bfd44d944-dlfcf   1/1     Running   0          2m33s
pod/nginx-deploy-6bfd44d944-rgmjn   1/1     Running   0          2m33s
pod/nginx-deploy-6bfd44d944-tpvfz   1/1     Running   0          2m33s

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP        15m
service/nodeport-svc   NodePort    10.96.145.18   <none>        80:30001/TCP   7s

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     3            3           2m33s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-6bfd44d944   3         3         3       2m33s
```

- Check the service:
```console
root@localhost:~# kubectl describe svc nodeport-svc
Name:                     nodeport-svc
Namespace:                default
Labels:                   env=demo
Annotations:              <none>
Selector:                 env=demo
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.96.145.18
IPs:                      10.96.145.18
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30001/TCP
Endpoints:                10.244.1.4:80,10.244.2.3:80,10.244.2.4:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

- Current state:
```console
root@localhost:~# kubectl get pod,deploy,svc --show-labels -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES   LABELS
pod/nginx-deploy-c95b4f658-gj6jl   1/1     Running   0          47s   10.244.2.4   lucky-luke-worker2   <none>           <none>            env=demo,pod-template-hash=c95b4f658,type=frontend
pod/nginx-deploy-c95b4f658-nc2qg   1/1     Running   0          47s   10.244.1.4   lucky-luke-worker    <none>           <none>            env=demo,pod-template-hash=c95b4f658,type=frontend
pod/nginx-deploy-c95b4f658-nfgrn   1/1     Running   0          47s   10.244.2.3   lucky-luke-worker2   <none>           <none>            env=demo,pod-template-hash=c95b4f658,type=frontend

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS        IMAGES   SELECTOR   LABELS
deployment.apps/nginx-deploy   3/3     3            3           47s   nginx-container   nginx    env=demo   env=demo

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE    SELECTOR   LABELS
service/kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP        21m    <none>     component=apiserver,provider=kubernetes
service/nodeport-svc   NodePort    10.96.145.18   <none>        80:30001/TCP   6m5s   env=demo   env=demo

```

- Check the `service` from our local:
```console
root@localhost:~# curl localhost:30001
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
---

### 2. Cluster IP
Because every `pod` which is restarted has got another `IP` address, we need to create a `clusterIP` service type. It's default `service` type in `Kubernetes`. We only need `port` and `targetPort` in its yaml configuration file.
`selector` section is related to `deployment` label.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hqwrgo0mr1edfzqm36hh.png)
(Photo from the video)

#### Implementation
```yaml
apiVersion: v1

kind: Service

metadata:
  name: clusterip-svc
  labels:
    env: demo

spec:
  selector:
      env: demo
  type:
    ClusterIP
  ports:
    - port: 80
      targetPort: 80

```

```console
root@localhost:~# kubectl apply -f clusterip.yaml
service/clusterip-svc created
root@localhost:~# kubectl get svc
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
clusterip-svc   ClusterIP   10.96.242.10   <none>        80/TCP         6s
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        67m
nodeport-svc    NodePort    10.96.145.18   <none>        80:30001/TCP   51m
root@localhost:~# kubectl get svc --show-labels -o wide
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE   SELECTOR   LABELS
clusterip-svc   ClusterIP   10.96.242.10   <none>        80/TCP         23s   env=demo   env=demo
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        67m   <none>     component=apiserver,provider=kubernetes
nodeport-svc    NodePort    10.96.145.18   <none>        80:30001/TCP   51m   env=demo   env=demo
root@localhost:~# kubectl describe svc clusterip-svc
Name:              clusterip-svc
Namespace:         default
Labels:            env=demo
Annotations:       <none>
Selector:          env=demo
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.242.10
IPs:               10.96.242.10
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.4:80,10.244.2.3:80,10.244.2.4:80
Session Affinity:  None
Events:            <none>
root@localhost:~# kubectl get endpoints
NAME            ENDPOINTS                                   AGE
clusterip-svc   10.244.1.4:80,10.244.2.3:80,10.244.2.4:80   3m25s
kubernetes      172.19.0.2:6443                             70m
nodeport-svc    10.244.1.4:80,10.244.2.3:80,10.244.2.4:80   54m

```
---

### 3. Load Balancer
`LoadBalancer` services are ideal for applications that need to handle high traffic volumes, such as web applications or APIs. With `LoadBalancer` services, we can access our application using a single IP address assigned to the load balancer.[source](https://www.baeldung.com/ops/kubernetes-service-types#3-loadbalancer-services)
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/18krfnomfxnktom6szfe.png)
(Photo from the video)

#### Implementation

```yaml
apiVersion: v1

kind: Service

metadata:
  name: loadbalancer-svc
  labels:
    env: demo

spec:
  selector:
      env: demo
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
```

```console
root@localhost:~# kubectl apply -f loadbalancer.yaml
service/loadbalancer-svc created
root@localhost:~# kubectl get svc
NAME               TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
clusterip-svc      ClusterIP      10.96.242.10    <none>        80/TCP         12m
kubernetes         ClusterIP      10.96.0.1       <none>        443/TCP        80m
loadbalancer-svc   LoadBalancer   10.96.153.185   <pending>     80:31076/TCP   7s
nodeport-svc       NodePort       10.96.145.18    <none>        80:30001/TCP   64m
root@localhost:~# kubectl get all
NAME                               READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-c95b4f658-gj6jl   1/1     Running   0          59m
pod/nginx-deploy-c95b4f658-nc2qg   1/1     Running   0          59m
pod/nginx-deploy-c95b4f658-nfgrn   1/1     Running   0          59m

NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/clusterip-svc      ClusterIP      10.96.242.10    <none>        80/TCP         13m
service/kubernetes         ClusterIP      10.96.0.1       <none>        443/TCP        80m
service/loadbalancer-svc   LoadBalancer   10.96.153.185   <pending>     80:31076/TCP   15s
service/nodeport-svc       NodePort       10.96.145.18    <none>        80:30001/TCP   64m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   3/3     3            3           59m

NAME                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-c95b4f658   3         3         3       59m

```
As we can see, `EXTERNAL-IP` for our `LoadBalancer` service type is `<pending>`. This is what we can provide an IP Address or DNS name for our users, and yer we don't have provision them in the service and it behaves as a `NodePort` service.
[For more info](https://kind.sigs.k8s.io/docs/user/loadbalancer/)

---

### 4. External Name
We actually map it to a DNS.
The below sample is from official documentation.[source](https://kubernetes.io/docs/concepts/services-networking/service/#externalname)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```
---

### 5. Creating `service` in command-line
Syntax:
`
kubectl create nodeport NAME [--tcp=port:targetPort] [--dry-run=server|client|none]
`
Example
```
kubectl create service nodeport myservice --node-port=31000 --tcp=3000:80
```
---

### 6. Useful links
- [Kubernetes Official](https://kubernetes.io/docs/reference/kubectl/quick-reference/)
- [Kubectl Command Cheatsheet](https://www.bluematador.com/learn/kubectl-cheatsheet)
- [Kubectl Reference Docs](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)





