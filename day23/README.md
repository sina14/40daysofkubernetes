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
```console
root@localhost:~# kubectl config set-credentials adam --client-key=adam.key --client-certificate=adam.crt
User "adam" set.
root@localhost:~# kubectl config set-context adam --cluster=kind-lucky-luke --user=adam
Context "adam" created.
root@localhost:~# kubectl config get-contexts
CURRENT   NAME              CLUSTER           AUTHINFO          NAMESPACE
          adam              kind-lucky-luke   adam
*         kind-lucky-luke   kind-lucky-luke   kind-lucky-luke

```

Switch to new context, `adam`
```console
root@localhost:~# kubectl config use-context adam
Switched to context "adam".
root@localhost:~# kubectl config get-contexts
CURRENT   NAME              CLUSTER           AUTHINFO          NAMESPACE
*         adam              kind-lucky-luke   adam
          kind-lucky-luke   kind-lucky-luke   kind-lucky-luke
root@localhost:~# kubectl auth whoami
ATTRIBUTE   VALUE
Username    adam
Groups      [system:authenticated]
root@localhost:~# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:39283
  name: kind-lucky-luke
contexts:
- context:
    cluster: kind-lucky-luke
    user: adam
  name: adam
- context:
    cluster: kind-lucky-luke
    user: kind-lucky-luke
  name: kind-lucky-luke
current-context: adam
kind: Config
preferences: {}
users:
- name: adam
  user:
    client-certificate: /root/adam.crt
    client-key: /root/adam.key
- name: kind-lucky-luke
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED

```

Check the permissions:
```console
root@localhost:~# kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
nginx-pod-3   1/1     Running   0          25s
root@localhost:~# kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "adam" cannot list resource "nodes" in API group "" at the cluster scope
```

check with `curl` and the keys
```console
root@localhost:~# curl https://127.0.0.1:39283/api/v1/namespaces/default/pods --key adam.key --cert adam.crt --cacert ca.crt
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "3003522"
  },
  "items": [
    {
      "metadata": {
        "name": "nginx-pod-3",
        "namespace": "default",
...
            "lastState": {},
            "ready": true,
            "restartCount": 0,
            "image": "docker.io/library/nginx:latest",
            "imageID": "docker.io/library/nginx@sha256:6af79ae5de407283dcea8b00d5c37ace95441fd58a8b1d2aa1ed93f5511bb18c",
            "containerID": "containerd://5ef66bb91400fd10cc3d9b9f0bfb08fd4d640df0f2c2834ec6598eb54edd4efb",
            "started": true
          }
        ],
        "qosClass": "BestEffort"
      }
    }
  ]
}
```

Let's check the validity of certification expired date of adam:
```console
root@localhost:~# openssl x509 -noout -dates -in adam.crt
notBefore=Jul 22 17:38:02 2024 GMT
notAfter=Aug  1 17:38:02 2024 GMT
```

We extend it for a year again with `openssl`
```console
root@localhost:~# openssl x509 -req -in adam.csr  -CA ca.crt -CAkey ca.key -CAcreateserial -out adam.crt -days 365
Certificate request self-signature ok
subject=CN = adam
root@localhost:~# openssl x509 -noout -dates -in adam.crt
notBefore=Jul 24 17:20:31 2024 GMT
notAfter=Jul 24 17:20:31 2025 GMT

```

New user instruction:
[source](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
```
1. openssl genrsa -out krishna.key 2048
2. openssl req -new -key krishna.key -out krishna.csr -subj "/CN=krishna"
3. change name, requests expired date in csr.yaml
4. cat krishna.csr | base64 >>> request
5. apply the file with kubectl
6. kubectl approve krishna
7. we have the role
8. we change the rolebinding and apply it
9. check can-i get pod as krishna
10. create crt file for krishna with certificate part of csr file and decode base64 and remove the new_line character
12. kubectl config set-credentials krishna with client-key, client-certificate
13. kubectl config set-context krishna with cluster name and user
14. kubectl config use-context krishan
15. check the premission

```

- Define Role and RoleBinding, Imperative way
```
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods

kubectl create rolebinding developer-binding-myuser --role=developer --user=myuser
```












