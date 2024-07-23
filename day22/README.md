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

