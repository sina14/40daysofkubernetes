## Day 23/40
# Kubernetes RBAC Explained - Role Based Access Control Kubernetes
[Video Link](https://www.youtube.com/watch?v=uGcDt7iNFkE)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section we are looking into Role Based Access Control, `RBAC`, authorization.

Simple check if I have access to get pods or not:
```console
root@localhost:~# kubectl auth whoami
ATTRIBUTE   VALUE
Username    kubernetes-admin
Groups      [kubeadm:cluster-admins system:authenticated]
root@localhost:~# kubectl auth can-i get pods
yes
```
Because I am kubernetes-admin, in the group cluster-admins, I'm authorized to get pods :)
Then, let's check `adam` has the right or not:
```console
root@localhost:~# kubectl auth can-i get pods --as adam
no
```
We have to grant him to do it, so we need to define `role` and `rolebinding` objects.

#### Role example
[source](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-example)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
Instead of `spec` we have `rules` in this object.

There are several API groups in Kubernetes:
- The core (also called legacy) group is found at REST path `/api/v1`. The core group is not specified as part of the `apiVersion` field, for example, `apiVersion: v1`.
- The named groups are at REST path `/apis/$GROUP_NAME/$VERSION` and use `apiVersion: $GROUP_NAME/$VERSION`
(for example, `apiVersion: batch/v1`). [source](https://kubernetes.io/docs/reference/using-api/#api-groups)























