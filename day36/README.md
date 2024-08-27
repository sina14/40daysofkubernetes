## Day 36/40
# Kubernetes Logging and Monitoring
[Video Link](https://www.youtube.com/watch?v=cNPyajLASms)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


Because we will start to troubleshooting some issues like:
- Application failures
- Control-plane issues
- Worker nodes issues
- Cluster components issues
And so on, we need to know about how `logging` or `monitoring` is in `Kubernetes`.


As we discussed, we know that `Kubernetes` doesn't come with an embedded monitoring tools, so we use `metrics-server` as add-on for getting some exposed metrics from our cluster.

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

```

There's also another daemon running on nodes, named `cAdvisor` which is collecting and aggregating the metrics from container runtime and then publish them to the `kubelet`.

But because only `kubelet` has authority to expose the data, and collect `pod` data as well, it sends them to the `metrics-server`.

`metrics-server` with the help of `metrics-api` can expose the data to `api-server` and when we call `kubectl top` we can reach out the data.

**Note** we may face an error when we apply the `metrics-server` manifest, so let's troubleshoot and fix the issue:

```sh
root@sinaops:~# kubectl get pods -n kube-system
NAME                              READY   STATUS    RESTARTS      AGE
coredns-7db6d8ff4d-2gdsx          1/1     Running   0             22h
coredns-7db6d8ff4d-tck9c          1/1     Running   0             22h
etcd-sinaops                      1/1     Running   0             22h
kube-apiserver-sinaops            1/1     Running   0             22h
kube-controller-manager-sinaops   1/1     Running   0             22h
kube-proxy-5t5bv                  1/1     Running   0             20h
kube-proxy-gt7vh                  1/1     Running   0             22h
kube-scheduler-sinaops            1/1     Running   0             22h
metrics-server-7ffbc6d68-9rw8z    0/1     Running   3 (18m ago)   22m
```
As we can see the `metrics-server` isn't ready yet.
```
NAME                              READY
metrics-server-7ffbc6d68-9rw8z    0/1
```

let's take a look at its logs:

```
...
E0827 16:09:23.312687       1 scraper.go:149] "Failed to scrape node" err="Get \"https://{JOLLY-NET-IP}:10250/metrics/resource\": tls: failed to verify certificate: x509: cannot validate certificate for {JOLLY-NET-IP} because it doesn't contain any IP SANs" node="jolly-net"
I0827 16:09:23.331929       1 server.go:191] "Failed probe" probe="metric-storage-ready" err="no metrics to serve"
...
```

And:

```
root@sinaops:~# kubectl describe pod metrics-server-7ffbc6d68-9rw8z -n kube-system
...
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  26m                  default-scheduler  Successfully assigned kube-system/metrics-server-7ffbc6d68-9rw8z to jolly-net
  Normal   Pulling    26m                  kubelet            Pulling image "registry.k8s.io/metrics-server/metrics-server:v0.7.1"
  Normal   Pulled     24m                  kubelet            Successfully pulled image "registry.k8s.io/metrics-server/metrics-server:v0.7.1" in 1.301s (2m22.567s including waiting). Image size: 68346568 bytes.
  Normal   Killing    23m (x2 over 23m)    kubelet            Container metrics-server failed liveness probe, will be restarted
  Warning  Unhealthy  23m (x4 over 23m)    kubelet            Readiness probe failed: Get "https://10.85.0.5:10250/readyz": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
...  
  Warning  Unhealthy  77s (x143 over 22m)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 500

...

```

So we're facing error code 500, because 

```
failed to verify certificate: x509: cannot validate certificate for {JOLLY-NET-IP} because it doesn't contain any IP SANs" node="jolly-net"

```
As it mentioned in the [github repository]() of `metrics-server`, 
> `Kubelet` certificate needs to be signed by cluster Certificate Authority (or disable certificate validation by passing `--kubelet-insecure-tls` to Metrics Server)
And we are going to edit the `deployment`.

```sh
root@sinaops:~# kubectl get deployment -n kube-system
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
coredns          2/2     2            2           22h
metrics-server   0/1     1            0           31m
root@sinaops:~# kubectl edit deployment metrics-server -n kube-system
deployment.apps/metrics-server edited

```

And in the `spec` of containers, add the option `--kubelet-insecure-tls`, then save and exit:

```yaml
...
    spec:
      containers:
      - args:
        - --kubelet-insecure-tls
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
```

Now, we are ready :)

```sh
root@sinaops:~# kubectl get pods -n kube-system -w
NAME                              READY   STATUS    RESTARTS   AGE
coredns-7db6d8ff4d-2gdsx          1/1     Running   0          22h
coredns-7db6d8ff4d-tck9c          1/1     Running   0          22h
etcd-sinaops                      1/1     Running   0          22h
kube-apiserver-sinaops            1/1     Running   0          22h
kube-controller-manager-sinaops   1/1     Running   0          22h
kube-proxy-5t5bv                  1/1     Running   0          20h
kube-proxy-gt7vh                  1/1     Running   0          22h
kube-scheduler-sinaops            1/1     Running   0          22h
metrics-server-8455d49879-5mqr2   1/1     Running   0          2m46s

```

We have to wait some time to it collects the metrics:

```sh
root@sinaops:~# kubectl top nodes
error: Metrics API not available
root@sinaops:~# kubectl top pods
error: Metrics API not available

```





































