## Day 19/40
# Kubernetes configmap and secret 
[Video Link](https://www.youtube.com/watch?v=Q9fHJLSyd7Q)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


We will look at `configMap` an `secret` in `Kubernetes` concepts in this section.
It's not a best practice to save `env`, key and value in yaml file of a workload such as pod yaml file.

> Kubernetes Secrets and ConfigMaps separate the configuration of individual container instances from the container image, reducing overhead and adding flexibility.

> Kubernetes has two types of objects that can inject configuration data into a container when it starts up.

[source](https://opensource.com/article/19/6/introduction-kubernetes-secrets-and-configmaps)

---

### Demo
Let's see the below `yaml` file, there's an env with name `FIRSTNAME` and value is `Piyush`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    env:
    - name: FIRSTNAME
      value: "Piyush"
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

Let's run and check the `env` inside the `pod`:
```console
root@localhost:~# kubectl apply -f day19-configmap.yaml
pod/myapp-pod created
root@localhost:~# kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          8s
root@localhost:~# kubectl exec -it myapp-pod -- sh
/ # echo $FIRSTNAME
Piyush
```

Now, we are going to do it with different ways as mentioned, `configMap`.
- Imperative:
```console
root@localhost:~# kubectl create cm app-cm --from-literal=FIRSTNAME=Piyush --from-literal=NEXT=Sina
configmap/app-cm created
root@localhost:~# kubectl get cm
NAME               DATA   AGE
app-cm             2      5s
kube-root-ca.crt   1      16d
root@localhost:~# kubectl describe cm app-cm
Name:         app-cm
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
FIRSTNAME:
----
Piyush
NEXT:
----
Sina

BinaryData
====

Events:  <none>
```
Then we need to inject to `yaml` file of our `pod`, and define specific block for each environment variable:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    env:
    - name: firstName
      valueFrom:
        configMapKeyRef:
          name: app-cm
          key: FIRSTNAME
    - name: next
      valueFrom:
        configMapKeyRef:
          name: app-cm
          key: NEXT
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```
Let's run and check:
```console
root@localhost:~# kubectl apply -f day19-configmap.yaml
pod/myapp-pod created
root@localhost:~# kubectl get pod
NAME        READY   STATUS    RESTARTS   AGE
myapp-pod   1/1     Running   0          36s
root@localhost:~# kubectl exec -it myapp-pod -- sh
/ # echo $firstName
Piyush
/ # echo $next
Sina
/ # 
```
```console
root@localhost:~# kubectl describe po/myapp-pod
Name:             myapp-pod
...
    Environment:
      firstName:  <set to the key 'FIRSTNAME' of config map 'app-cm'>  Optional: false
      next:       <set to the key 'NEXT' of config map 'app-cm'>       Optional: false
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-
...
```

- Declarative:
Let's create `yaml` file from `dry-run` option:
```console
root@localhost:~# kubectl create cm app-cm --from-literal=FIRSTNAME=Piyush --from-literal=NEXT=Sina --dry-run=client -o yaml > day19-cm.yaml
```
The `yaml` file would be below:
```yaml
apiVersion: v1
data:
  FIRSTNAME: Piyush
  NEXT: Sina
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: app-cm
```
---

Sample for `cm`:
```yaml
apiVersion: v1
data:
  FIRSTNAME: Piyush
  NEXT: Sina
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: app-cm
```

Sample for `pod`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: app-cm
  restartPolicy: Never
```

Let's check:
```console
root@localhost:~# kubectl get cm,pod
NAME                         DATA   AGE
configmap/app-cm             2      5m53s
configmap/kube-root-ca.crt   1      16d

NAME                READY   STATUS      RESTARTS   AGE
pod/dapi-test-pod   0/1     Completed   0          4m15s
```
```console
root@localhost:~# kubectl describe pod dapi-test-pod
Name:             dapi-test-pod
...
    Environment Variables from:
      app-cm      ConfigMap  Optional: false

...
```




















