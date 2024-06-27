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
As you can see the second `pod` is running on worker1 `node`. Also instead of using *create* with `kubectl`, we can use *apply* with `kubectl`.


### 3. Update and edit directly an object on the cluster
For instance we need to change the version of `nginx` which is running with the latest version to 1.18. We run the command and change the version in front of `image` key and then save it.
```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2024-06-27T16:48:31Z"
  labels:
    run: nginx-pod
  name: nginx-pod
  namespace: default
  resourceVersion: "9983"
  uid: 6d0bd9ee-9f92-45b4-9416-87e02568a435
spec:
  containers:
  - image: nginx:1.18
    imagePullPolicy: Always
...
```
```console
root@localhost:~# kubectl edit pod nginx-pod
pod/nginx-pod edited
```

for confirmation and see what happened, you can run the below command and see the logs and container version that is already is running.

```console
root@localhost:~# kubectl describe pod nginx-pod
Name:             nginx-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             lucky-luke-worker2/172.19.0.2
Start Time:       Thu, 27 Jun 2024 16:48:31 +0000
Labels:           run=nginx-pod
Annotations:      <none>
Status:           Running
IP:               10.244.2.2
IPs:
  IP:  10.244.2.2
Containers:
  nginx-pod:
    Container ID:   containerd://b9ff29ff39e7446d9ad8ab9a60fe8f29d8c4ed4ceb35c9628bc692d228bd096c
    Image:          nginx:1.18
    Image ID:       docker.io/library/nginx@sha256:e90ac5331fe095cea01b121a3627174b2e33e06e83720e9a934c7b8ccc9c55a0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 27 Jun 2024 17:40:58 +0000
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 27 Jun 2024 16:48:41 +0000
      Finished:     Thu, 27 Jun 2024 17:40:51 +0000
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-j4sf7 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-j4sf7:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age                From               Message
  ----    ------     ----               ----               -------
  Normal  Scheduled  52m                default-scheduler  Successfully assigned default/nginx-pod to lucky-luke-worker2
  Normal  Pulling    52m                kubelet            Pulling image "nginx:latest"
  Normal  Pulled     52m                kubelet            Successfully pulled image "nginx:latest" in 8.319s (8.319s including waiting). Image size: 71010466 bytes.
  Normal  Killing    17s                kubelet            Container nginx-pod definition changed, will be restarted
  Normal  Pulling    17s                kubelet            Pulling image "nginx:1.18"
  Normal  Created    10s (x2 over 52m)  kubelet            Created container nginx-pod
  Normal  Started    10s (x2 over 52m)  kubelet            Started container nginx-pod
  Normal  Pulled     10s                kubelet            Successfully pulled image "nginx:1.18" in 6.882s (6.882s including waiting). Image size: 53630764 bytes.
```

### 4. Interaction with the container
 ```console
root@localhost:~# kubectl exec -it nginx-pod -- sh
# pwd
/
command terminated with exit code 127
root@localhost:~# kubectl get pods
NAME          READY   STATUS    RESTARTS        AGE
nginx-pod     1/1     Running   1 (8m19s ago)   60m
nginx-pod-2   1/1     Running   0  
```
Terminating with `Ctrl`+`D`

### 5. Create `yaml` file of a `pod` in `dry-run` mode
```console
root@localhost:~# kubectl run nginx-pod-3 --image=nginx:latest --dry-run=client -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx-pod3
  name: nginx-pod3
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod3
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

```
You can direct the result into a new `yaml` file and make your own change.
```console
root@localhost:~# kubectl run nginx-pod-3 --image=nginx:latest --dry-run=client -o yaml > nginx-pod-3.yaml
```
**Note** in `dry-run` step nothing is creates and you can use the `dry-run` step to check your manifests before deployment.

### 6. Get more details of running pods
```console
root@localhost:~# kubectl get pods -o wide
NAME          READY   STATUS    RESTARTS      AGE   IP           NODE                 NOMINATED NODE   READINESS GATES
nginx-pod     1/1     Running   1 (19m ago)   71m   10.244.2.2   lucky-luke-worker2   <none>           <none>
nginx-pod-2   1/1     Running   0             43m   10.244.1.2   lucky-luke-worker    <none>           <none>
root@localhost:~# kubectl get pods --show-labels
NAME          READY   STATUS    RESTARTS      AGE   LABELS
nginx-pod     1/1     Running   1 (21m ago)   73m   run=nginx-pod
nginx-pod-2   1/1     Running   0             44m   env=demo,type=frontend
```
Labels are good for grouping our pods.

---

### Task 3
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: test
  name: redis
spec:
  containers:
  - image: rediss
    name: redis
```

```console
root@localhost:~# kubectl apply -f task3.yaml
pod/redis created
root@localhost:~# kubectl get pods
NAME          READY   STATUS         RESTARTS      AGE
nginx-pod     1/1     Running        1 (36m ago)   89m
nginx-pod-2   1/1     Running        0             60m
redis         0/1     ErrImagePull   0             12s
root@localhost:~# kubectl describe pod redis
Name:             redis
Namespace:        default
Priority:         0
Service Account:  default
Node:             lucky-luke-worker2/172.19.0.2
Start Time:       Thu, 27 Jun 2024 18:17:24 +0000
Labels:           app=test
Annotations:      <none>
Status:           Pending
IP:               10.244.2.3
IPs:
  IP:  10.244.2.3
Containers:
  redis:
    Container ID:
    Image:          rediss
    Image ID:
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qks6m (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       False
  ContainersReady             False
  PodScheduled                True
Volumes:
  kube-api-access-qks6m:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  94s                default-scheduler  Successfully assigned default/redis to lucky-luke-worker2
  Normal   BackOff    17s (x5 over 92s)  kubelet            Back-off pulling image "rediss"
  Warning  Failed     17s (x5 over 92s)  kubelet            Error: ImagePullBackOff
  Normal   Pulling    3s (x4 over 93s)   kubelet            Pulling image "rediss"
  Warning  Failed     2s (x4 over 92s)   kubelet            Failed to pull image "rediss": failed to pull and unpack image "docker.io/library/rediss:latest": failed to resolve reference "docker.io/library/rediss:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
  Warning  Failed     2s (x4 over 92s)   kubelet            Error: ErrImagePull
root@localhost:~# kubectl edit pod redis
pod/redis edited
root@localhost:~# kubectl get pods
NAME          READY   STATUS             RESTARTS      AGE
nginx-pod     1/1     Running            1 (38m ago)   90m
nginx-pod-2   1/1     Running            0             62m
redis         0/1     ImagePullBackOff   0             2m5s
root@localhost:~# kubectl get pods
NAME          READY   STATUS    RESTARTS      AGE
nginx-pod     1/1     Running   1 (38m ago)   91m
nginx-pod-2   1/1     Running   0             62m
redis         1/1     Running   0             2m10s

```
As we see the image is not valid and we edit to correct image name and the problem solved!





