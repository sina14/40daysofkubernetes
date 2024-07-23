## Day 22/40
# Kubernetes Authentication and Authorization Simply Explained
[Video Link](https://www.youtube.com/watch?v=P0bogYEyfeI)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)



We're looking at `authorization`, `kubeconfig` and some other concepts.
Actually when you run command `kubectl get pods`, before you get the result, you will be authenticate and check if you're authorized and permit to run this command in a `kubernetes` cluster.

Because in the background, some data is sent to the cluster, you are or not authorized to run commands in a cluster.
We are passing these options with a config file called `kubeconfig`.

The actual command is like:
```sh
root@localhost:~# kubectl get nodes --kubeconfig .kube/config
NAME                       STATUS   ROLES           AGE   VERSION
lucky-luke-control-plane   Ready    control-plane   22d   v1.30.0
lucky-luke-worker          Ready    <none>          22d   v1.30.0
lucky-luke-worker2         Ready    <none>          22d   v1.30.0
```

The details in `kubeconfig` file include:
- certificate-authority-data
- server
- contexts
- client-key-data
and so on.
the `context` is nothing but the combination of `user` and `cluster`.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u29nqqxp9dyuej5gb3mu.png)
(Photo from the video)

```yaml
...
contexts:
- context:
    cluster: kind-lucky-luke
    user: kind-lucky-luke
  name: kind-lucky-luke
...
```

### What is Authentication and What is Authorization

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rlqno4jby3sy9ghz4u6s.png)
(Photo from the video)

**ABAC**, Attribute-Based Access Control, defines an access control paradigm whereby access rights are granted to users through the use of policies which combine attributes together.[source](https://kubernetes.io/docs/reference/access-authn-authz/abac/)

**RBAC**, Role-based access control, is a method of regulating access to computer or network resources based on the roles of individual users within your organization.[source](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

**NODE** authorization is a special-purpose authorization mode that specifically authorizes API requests made by kubelets.[source](https://kubernetes.io/docs/reference/access-authn-authz/node/)

A **WebHook** is an HTTP callback: an HTTP POST that occurs when something happens; a simple event-notification via HTTP POST. A web application implementing WebHooks will POST a message to a URL when certain things happen.
When specified, mode `Webhook` causes Kubernetes to query an outside REST service when determining user privileges.[source](https://kubernetes.io/docs/reference/access-authn-authz/webhook/)

---

As we are using a `kind` cluster, we cannot ssh to our node, because it's an docker container. So we can handle it with `exec` command.

```console
root@localhost:~# docker ps | grep control-plane
f791fa85c269   kindest/node:v1.30.0   "/usr/local/bin/entr…"   3 weeks ago    Up 11 hours   0.0.0.0:30001->30001/tcp, 127.0.0.1:39283->6443/tcp                                            lucky-luke-control-plane
root@localhost:~# docker exec -it lucky-luke-control-plane bash
root@lucky-luke-control-plane:/# cd /etc/kubernetes/manifests/
root@lucky-luke-control-plane:/etc/kubernetes/manifests# ls -l
total 16
-rw------- 1 root root 2418 Jul 23 06:26 etcd.yaml
-rw------- 1 root root 3896 Jul 23 06:26 kube-apiserver.yaml
-rw------- 1 root root 3434 Jul 23 06:26 kube-controller-manager.yaml
-rw------- 1 root root 1463 Jul 23 06:26 kube-scheduler.yaml
```

If we see the `kube-apiserver.yaml` file, we can see many options that have passed to starting the service command such as:
```yaml
...
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=172.19.0.4
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
...
```
The `authorization-mode` is `Node` and `RBAC`.
**Note** Defaults to `AlwaysAllow` if `--authorization-config` is not used. 
[Here](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/#options) for more info about Other options.

---

The default directory of all certificate which is used by `kube-apiserver` is:
```console
root@lucky-luke-control-plane:~# ls -l /etc/kubernetes/pki/
total 60
-rw-r--r-- 1 root root 1123 Jul  1 16:16 apiserver-etcd-client.crt
-rw------- 1 root root 1675 Jul  1 16:16 apiserver-etcd-client.key
-rw-r--r-- 1 root root 1176 Jul  1 16:16 apiserver-kubelet-client.crt
-rw------- 1 root root 1679 Jul  1 16:16 apiserver-kubelet-client.key
-rw-r--r-- 1 root root 1334 Jul 23 06:26 apiserver.crt
-rw------- 1 root root 1675 Jul 23 06:26 apiserver.key
-rw-r--r-- 1 root root 1107 Jul  1 16:16 ca.crt
-rw------- 1 root root 1675 Jul  1 16:16 ca.key
drwxr-xr-x 2 root root 4096 Jul  1 16:16 etcd
-rw-r--r-- 1 root root 1123 Jul  1 16:16 front-proxy-ca.crt
-rw------- 1 root root 1679 Jul  1 16:16 front-proxy-ca.key
-rw-r--r-- 1 root root 1119 Jul  1 16:16 front-proxy-client.crt
-rw------- 1 root root 1675 Jul  1 16:16 front-proxy-client.key
-rw------- 1 root root 1679 Jul  1 16:16 sa.key
-rw------- 1 root root  451 Jul  1 16:16 sa.pub
```

There are multiple pair of certificates and key for apiserver, because sometimes it acts as a server and sometimes it acts like a client.

















