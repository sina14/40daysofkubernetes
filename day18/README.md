## Day 18/40
# Kubernetes Probes Explained | Liveness vs Readiness vs Startup Probes
[Video Link](https://www.youtube.com/watch?v=x2e6pIBLKzw)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


It's time to explain the **Health Probes** like:
- Liveness Probe
- Readiness Probe
- Startup Probe
and so on.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ijmncxbp9ohf6revodjk.png)
(Photo from the video)

---
In Kubernetes, a probe is a mechanism used to determine the health and readiness of a container or application running within a pod. Probes are defined in the pod specification and are performed periodically to ensure the proper functioning of the application.
- **Liveness Probe**: A liveness probe determines if a container is still running and functioning correctly.
- **Readiness Probe**: A readiness probe determines if a container is ready to receive incoming network traffic and serve requests. It ensures that the container has completed its initialization process and is prepared to handle requests.
- **Startup Probe**: A startup probe is used to determine if a container's application has started successfully. 

[source](https://kubeops.net/blog/kubernetes-probes)

---

### Demo
- Sample1: liveness command
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -f /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat 
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

```console
root@localhost:~# kubectl apply -f day18-liveness-command.yaml
pod/liveness-exec created
root@localhost:~# kubectl get pod --watch
NAME            READY   STATUS    RESTARTS   AGE
liveness-exec   1/1     Running   0          11s
liveness-exec   1/1     Running   1 (0s ago)   76s
liveness-exec   1/1     Running   2 (2s ago)   2m33s
```
Logs of the `pod`:
```
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  3m37s                default-scheduler  Successfully assigned default/liveness-exec to lucky-luke-worker
  Normal   Pulled     3m36s                kubelet            Successfully pulled image "registry.k8s.io/busybox" in 1.08s (1.08s including waiting). Image size: 1144547 bytes.
  Normal   Pulled     2m22s                kubelet            Successfully pulled image "registry.k8s.io/busybox" in 367ms (367ms including waiting). Image size: 1144547 bytes.
  Normal   Pulling    67s (x3 over 3m37s)  kubelet            Pulling image "registry.k8s.io/busybox"
  Normal   Created    67s (x3 over 3m36s)  kubelet            Created container liveness
  Normal   Pulled     67s                  kubelet            Successfully pulled image "registry.k8s.io/busybox" in 503ms (505ms including waiting). Image size: 1144547 bytes.
  Normal   Started    66s (x3 over 3m36s)  kubelet            Started container liveness
  Warning  Unhealthy  22s (x9 over 3m2s)   kubelet            Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
  Normal   Killing    22s (x3 over 2m52s)  kubelet            Container liveness failed liveness probe, will be restarted
```

---
- Sample2: liveness http and readiness http
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello
spec:
  containers:
  - name: liveness
    image: registry.k8s.io/e2e-test-images/agnhost:2.40
    args:
    - liveness
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 10
```

```console
root@localhost:~# kubectl apply -f day18-liveness-http.yaml
pod/hello created
root@localhost:~# kubectl get pod
NAME    READY   STATUS    RESTARTS   AGE
hello   0/1     Running   0          10s
root@localhost:~# kubectl get pod --watch
NAME    READY   STATUS    RESTARTS   AGE
hello   0/1     Running   0          16s
hello   0/1     Running   1 (1s ago)   22s
hello   0/1     Running   2 (1s ago)   40s
```

```
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  2m11s               default-scheduler  Successfully assigned default/hello to lucky-luke-worker
  Normal   Pulling    2m11s               kubelet            Pulling image "registry.k8s.io/e2e-test-images/agnhost:2.40"
  Normal   Pulled     2m8s                kubelet            Successfully pulled image "registry.k8s.io/e2e-test-images/agnhost:2.40" in 3.022s (3.022s including waiting). Image size: 51155161 bytes.
  Normal   Created    92s (x3 over 2m8s)  kubelet            Created container liveness
  Normal   Started    92s (x3 over 2m7s)  kubelet            Started container liveness
  Warning  Unhealthy  92s (x3 over 111s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 500
  Normal   Pulled     92s (x2 over 110s)  kubelet            Container image "registry.k8s.io/e2e-test-images/agnhost:2.40" already present on machine
  Warning  Unhealthy  74s (x9 over 116s)  kubelet            Liveness probe failed: HTTP probe failed with statuscode: 500
  Normal   Killing    74s (x3 over 110s)  kubelet            Container liveness failed liveness probe, will be restarted
```

---
- Sample3: liveness tcp
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tcp-pod
  labels:
    app: tcp-pod
spec:
  containers:
  - name: goproxy
    image: registry.k8s.io/goproxy:0.1
    ports:
    - containerPort: 8080
    livenessProbe:
      tcpSocket:
        port: 3000
      initialDelaySeconds: 10
      periodSeconds: 5
```

```console
root@localhost:~# kubectl apply -f day18-liveness-tcp.yaml
pod/tcp-pod created
root@localhost:~# kubectl get pod --watch
NAME      READY   STATUS             RESTARTS        AGE
hello     0/1     CrashLoopBackOff   9 (2m13s ago)   15m
tcp-pod   1/1     Running            0               10s
tcp-pod   1/1     Running            1 (1s ago)      27s
tcp-pod   1/1     Running            2 (1s ago)      47s
tcp-pod   1/1     Running            3 (0s ago)      66s
hello     0/1     Terminating        9 (3m18s ago)   16m
hello     0/1     Terminating        9               16m
hello     0/1     Terminating        9               16m
hello     0/1     Terminating        9               16m
hello     0/1     Terminating        9               16m
hello     0/1     Terminating        9               16m
tcp-pod   1/1     Running            4 (0s ago)      86s

```

```
Events:
  Type     Reason     Age                 From               Message
  ----     ------     ----                ----               -------
  Normal   Scheduled  111s                default-scheduler  Successfully assigned default/tcp-pod to lucky-luke-worker
  Normal   Pulling    111s                kubelet            Pulling image "registry.k8s.io/goproxy:0.1"
  Normal   Pulled     110s                kubelet            Successfully pulled image "registry.k8s.io/goproxy:0.1" in 695ms (695ms including waiting). Image size: 1698862 bytes.
  Normal   Created    46s (x4 over 110s)  kubelet            Created container goproxy
  Normal   Started    46s (x4 over 110s)  kubelet            Started container goproxy
  Warning  Unhealthy  46s (x9 over 96s)   kubelet            Liveness probe failed: dial tcp 10.244.1.35:3000: connect: connection refused
  Normal   Killing    46s (x3 over 86s)   kubelet            Container goproxy failed liveness probe, will be restarted
  Normal   Pulled     46s (x3 over 86s)   kubelet            Container image "registry.k8s.io/goproxy:0.1" already present on machine
```

- **Note** As you can see in the part of running `pod` yaml file, there are some option for `livenessProbe`
- **failureThreshold: 3** which says after 3 failed tests, it can be assumed as a failuer and each test perfomes after `periodSeconds`.
- **successThreshold: 1** which says only after one success test, it can be assumed as successful pod.

```yaml
...
spec:
  containers:
  - image: registry.k8s.io/goproxy:0.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 3
      initialDelaySeconds: 10
      periodSeconds: 5
      successThreshold: 1
      tcpSocket:
        port: 3000
      timeoutSeconds: 1
...
```

- **Note** We can see some tolerations added to the `pod` like:
```yaml
...
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
...
```
These are automatically added by `Kubernetes` and if node get this taint, `pod` will be evicted from this `node` and move to another `node`.

**node.kubernetes.io/not-ready**
Type: Taint
Example: node.kubernetes.io/not-ready: "NoExecute"
Used on: Node
The Node controller detects whether a Node is ready by monitoring its health and adds or removes this taint accordingly.
[source](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-not-ready)

**node.kubernetes.io/unreachable**
Type: Taint
Example: node.kubernetes.io/unreachable: "NoExecute"
Used on: Node
The Node controller adds the taint to a Node corresponding to the NodeCondition Ready being Unknown.
[source](https://kubernetes.io/docs/reference/labels-annotations-taints/#node-kubernetes-io-unreachable)

---







