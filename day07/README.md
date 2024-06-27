## Day 7/40
# Pod In Kubernetes Explained | Imperative VS Declarative Way | YAML Tutorial
[Video Link](https://www.youtube.com/watch?v=_f9ql2Y5Xcc)
@piyushsachdeva 

There are 2 ways of configuration and deploying a workload such as `pod` in `Kubernetes`:
1. Imperative
2. Declarative

#### Imperative configuration
Which means that to describe the configuration of the resource, you execute a command from a terminal’s command prompt. (using `kubectl` for instance)

#### Declarative configuration
Which means that you create a file that describes the configuration for the particular resource and then apply the content of the file to the `Kubernetes` cluster. (using `yaml` file and `json` format)

---
**Note**
If you need a playground you can [Play with `Kubernetes`](https://labs.play-with-k8s.com/)

At first you have to create your environment with the instruction as they mention:
```
This is a sandbox environment. Using personal credentials
 is HIGHLY! discouraged. Any consequences of doing so, are
 completely the user's responsibilites.

 You can bootstrap a cluster as follows:

 1. Initializes cluster master node:

 kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16


 2. Initialize cluster networking:

 kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml


 3. (Optional) Create an nginx deployment:

 kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/application/nginx-app.yaml


                          The PWK team.
```

```console
[node1 ~]$ kubectl get nodes
NAME    STATUS   ROLES           AGE   VERSION
node1   Ready    control-plane   32s   v1.27.2
[node1 ~]$ kubectl config get-contexts 
CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
*         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin  
```
---

### 1. Create `pod` with imperative way
```console
root@localhost:~# kubectl get nodes
NAME                       STATUS   ROLES           AGE   VERSION
lucky-luke-control-plane   Ready    control-plane   89m   v1.30.0
lucky-luke-worker          Ready    <none>          88m   v1.30.0
lucky-luke-worker2         Ready    <none>          88m   v1.30.0
root@localhost:~# kubectl run nginx-pod --image=nginx:latest
pod/nginx-pod created
root@localhost:~# kubectl get pods
NAME        READY   STATUS              RESTARTS   AGE
nginx-pod   0/1     ContainerCreating   0          8s
root@localhost:~# kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          21s
root@localhost:~# kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-pod   1/1     Running   0          43s   10.244.2.2   lucky-luke-worker2   <none>           <none>
```
As you can see, our nginx-pod is running on worker2 `node` of our `cluster`, and it's containing one `container` running out of one.

### 2. Create `pod` with declarative way
Firstly, we need to create a `yaml` file with our desired options.
Read more about [`yaml`](https://spacelift.io/blog/yaml)
```console
root@localhost:~# touch day07-yaml.yaml
```
The starting content:
```yaml
# this is a sample file
#
apiVersion:

kind:

metadata:

spec:

```
for more information about it you can run the below command:
```console
root@localhost:~# kubectl explain pod
KIND:       Pod
VERSION:    v1

DESCRIPTION:
    Pod is a collection of containers that can run on a host. This resource is
    created by clients and scheduled onto hosts.

FIELDS:
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind  <string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata      <ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec  <PodSpec>
    Specification of the desired behavior of the pod. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

  status        <PodStatus>
    Most recently observed status of the pod. This data may not be up to date.
    Populated by the system. Read-only. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

```

final version of our `yaml` file:
```yaml
# this is a sample file
#
apiVersion: v1

kind: Pod

metadata:
  name: nginx-pod-2
  labels:
    env: demo
    type: frontend

spec:
  containers:
    - name: nginx-container
      image: nginx
      ports:
        - containerPort: 80
```
Create with `kubectl`
```console
root@localhost:~# kubectl create -f day07-yaml.yaml
pod/nginx-pod-2 created
root@localhost:~# kubectl get pods
NAME          READY   STATUS              RESTARTS   AGE
nginx-pod     1/1     Running             0          28m
nginx-pod-2   0/1     ContainerCreating   0          8s
root@localhost:~# kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
nginx-pod     1/1     Running   0          28m
nginx-pod-2   1/1     Running   0          16s
root@localhost:~# kubectl get pods -o wide
NAME          READY   STATUS    RESTARTS   AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-pod     1/1     Running   0          28m   10.244.2.2   lucky-luke-worker2   <none>           <none>
nginx-pod-2   1/1     Running   0          32s   10.244.1.2   lucky-luke-worker    <none>           <none>
```
As you can see the second `pod` is running on worker1 `node`
