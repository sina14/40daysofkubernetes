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

Let's check the all `role` in our cluster:
```console
root@localhost:~# kubectl get roles -A
NAMESPACE     NAME                                               CREATED AT
kube-public   kubeadm:bootstrap-signer-clusterinfo               2024-07-01T16:17:07Z
kube-public   system:controller:bootstrap-signer                 2024-07-01T16:17:06Z
kube-system   extension-apiserver-authentication-reader          2024-07-01T16:17:06Z
kube-system   kube-proxy                                         2024-07-01T16:17:08Z
kube-system   kubeadm:kubelet-config                             2024-07-01T16:17:07Z
kube-system   kubeadm:nodes-kubeadm-config                       2024-07-01T16:17:07Z
kube-system   leader-locking-nfs-client-nfs-client-provisioner   2024-07-06T07:07:02Z
kube-system   system::leader-locking-kube-controller-manager     2024-07-01T16:17:06Z
kube-system   system::leader-locking-kube-scheduler              2024-07-01T16:17:06Z
kube-system   system:controller:bootstrap-signer                 2024-07-01T16:17:06Z
kube-system   system:controller:cloud-provider                   2024-07-01T16:17:06Z
kube-system   system:controller:token-cleaner                    2024-07-01T16:17:06Z

```
And rolebindings:
```console
root@localhost:~# kubectl get rolebinding -A
NAMESPACE     NAME                                                ROLE                                                    AGE
kube-public   kubeadm:bootstrap-signer-clusterinfo                Role/kubeadm:bootstrap-signer-clusterinfo               23d
kube-public   system:controller:bootstrap-signer                  Role/system:controller:bootstrap-signer                 23d
kube-system   kube-proxy                                          Role/kube-proxy                                         23d
kube-system   kubeadm:kubelet-config                              Role/kubeadm:kubelet-config                             23d
kube-system   kubeadm:nodes-kubeadm-config                        Role/kubeadm:nodes-kubeadm-config                       23d
kube-system   leader-locking-nfs-client-nfs-client-provisioner    Role/leader-locking-nfs-client-nfs-client-provisioner   18d
kube-system   system::extension-apiserver-authentication-reader   Role/extension-apiserver-authentication-reader          23d
kube-system   system::leader-locking-kube-controller-manager      Role/system::leader-locking-kube-controller-manager     23d
kube-system   system::leader-locking-kube-scheduler               Role/system::leader-locking-kube-scheduler              23d
kube-system   system:controller:bootstrap-signer                  Role/system:controller:bootstrap-signer                 23d
kube-system   system:controller:cloud-provider                    Role/system:controller:cloud-provider                   23d
kube-system   system:controller:token-cleaner                     Role/system:controller:token-cleaner                    23d
```

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
Instead of `spec` we have `rules` in this object. `verbs` is access and when we set blank in the `apiGroups` it indicates the **core** API group.

There are several API groups in Kubernetes:
- The **core** (also called legacy) group is found at REST path `/api/v1`. The core group is not specified as part of the `apiVersion` field, for example, `apiVersion: v1`.
- The **named** groups are at REST path `/apis/$GROUP_NAME/$VERSION` and use `apiVersion: $GROUP_NAME/$VERSION`
(for example, `apiVersion: batch/v1`).
[source](https://kubernetes.io/docs/reference/using-api/#api-groups)

Let's apply the `role` in our cluster:
```console
root@localhost:~# kubectl get role
No resources found in default namespace.
root@localhost:~# kubectl apply -f day23-role.yaml
role.rbac.authorization.k8s.io/pod-reader created
root@localhost:~# kubectl get roles
NAME         CREATED AT
pod-reader   2024-07-24T16:27:15Z
root@localhost:~# kubectl describe role pod-reader
Name:         pod-reader
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get watch list]

```

It still needs to define to which user, this `role` should be applied, so we will define `rolebinding` which is binding roles to users.

#### RoleBinding example

[source](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-example)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "adam" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: adam # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
Apply the `yaml` file:
```console
root@localhost:~# kubectl get rolebinding
No resources found in default namespace.
root@localhost:~# kubectl apply -f day23-rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/read-pods created
root@localhost:~# kubectl get rolebinding read-pods
NAME        ROLE              AGE
read-pods   Role/pod-reader   37s
root@localhost:~# kubectl describe rolebinding read-pods
Name:         read-pods
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  pod-reader
Subjects:
  Kind  Name  Namespace
  ----  ----  ---------
  User  adam
```

And finally:
```sh
root@localhost:~# kubectl auth can-i get pod --as adam
yes
```

Then we are going to log in as user `adam` and get pods:





















