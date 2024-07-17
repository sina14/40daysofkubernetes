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











