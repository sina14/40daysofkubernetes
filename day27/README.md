## Day 27/40
# Setup a Multi Node Kubernetes Cluster Using Kubeadm
[Video Link](https://www.youtube.com/watch?v=WcdMC3Lj4tU)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


We're going to setup a multi-nodes `Kubernetes` cluster with [`kubeadm`](https://kubernetes.io/docs/reference/setup-tools/kubeadm/).
[Installation Guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Before we begin, please take a look at the below flowchart and answer some questions to identifying the purpose of installation.

![purpose of installation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/77qh17fflz15clkf7xnj.png)

(Photo from the video) 

To know about which ports and protocols we need in this process please look at the [documentation](https://kubernetes.io/docs/reference/networking/ports-and-protocols/)

#### Here's what we need:

- Control plane(s)

| Protocol | Direction | Port Range | Purpose | Used By |
| -------- | -------- | -------- | -------- | -------- |
| TCP | Inbound | 6443 | Kubernetes API server | All |
| TCP | Inbound | 2379-2380 | etcd server client API | kube-apiserver, etcd |
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10259 | kube-scheduler | Self |
| TCP | Inbound | 10257 | kube-controller-manager | Self |


- Worker Node(s)

| Protocol | Direction | Port Range | Purpose | Used By |
| -------- | -------- | -------- | -------- | -------- |
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10256 | kube-proxy | Self, Load balancers |
| TCP | Inbound | 30000-32767 | NodePort Services† | All |

**Note** default `NodePort` range is 30000-32767.

![ports overview](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/45phbopcu024cirj9394.png)

(Photo from the video)

After opening the neccessary ports and protocols, we have the below steps to continue:

![Installation steps](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tlv14o5f12k4obfgdqf5.png)

(Photo from the video)

---

## 1. Setup the Control plane(s)

**Note** I had some problemS with the instruction that's introduced in the video and I followed another one [here](https://devopscube.com/setup-kubernetes-cluster-kubeadm/)

```console
root@hirmand:~# kubeadm init --apiserver-advertise-address=192.168.1.201  --apiserver-cert-extra-sans=192.168.1.201  --pod-network-cidr=$POD_CIDR --node-name $NODENAME --ignore-preflight-errors Swap

...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.1.201:6443 --token XXXX \
        --discovery-token-ca-cert-hash sha256:XXXX
root@hirmand:~#
root@hirmand:~# mkdir -p $HOME/.kube
root@hirmand:~# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
cp: overwrite '/root/.kube/config'? y
root@hirmand:~# sudo chown $(id -u):$(id -g) $HOME/.kube/config
root@hirmand:~# kubectl get nodes
NAME      STATUS   ROLES           AGE     VERSION
hirmand   Ready    control-plane   6m45s   v1.29.6
root@hirmand:~# kubectl get po -n kube-system
NAME                              READY   STATUS    RESTARTS   AGE
coredns-76f75df574-cpthh          1/1     Running   0          2m44s
coredns-76f75df574-jx2tr          1/1     Running   0          2m44s
etcd-hirmand                      1/1     Running   0          2m58s
kube-apiserver-hirmand            1/1     Running   0          2m56s
kube-controller-manager-hirmand   1/1     Running   0          2m56s
kube-proxy-v24nc                  1/1     Running   0          2m44s
kube-scheduler-hirmand            1/1     Running   0          2m56s

```

>You verify all the cluster component health statuses using the following command.
```console
root@hirmand:~# kubectl get --raw='/readyz?verbose'
[+]ping ok
[+]log ok
[+]etcd ok
[+]etcd-readiness ok
[+]informer-sync ok
[+]poststarthook/start-kube-apiserver-admission-initializer ok
[+]poststarthook/generic-apiserver-start-informers ok
[+]poststarthook/priority-and-fairness-config-consumer ok
[+]poststarthook/priority-and-fairness-filter ok
[+]poststarthook/storage-object-count-tracker-hook ok
[+]poststarthook/start-apiextensions-informers ok
[+]poststarthook/start-apiextensions-controllers ok
[+]poststarthook/crd-informer-synced ok
[+]poststarthook/start-service-ip-repair-controllers ok
[+]poststarthook/rbac/bootstrap-roles ok
[+]poststarthook/scheduling/bootstrap-system-priority-classes ok
[+]poststarthook/priority-and-fairness-config-producer ok
[+]poststarthook/start-system-namespaces-controller ok
[+]poststarthook/bootstrap-controller ok
[+]poststarthook/start-cluster-authentication-info-controller ok
[+]poststarthook/start-kube-apiserver-identity-lease-controller ok
[+]poststarthook/start-kube-apiserver-identity-lease-garbage-collector ok
[+]poststarthook/start-legacy-token-tracking-controller ok
[+]poststarthook/aggregator-reload-proxy-client-cert ok
[+]poststarthook/start-kube-aggregator-informers ok
[+]poststarthook/apiservice-registration-controller ok
[+]poststarthook/apiservice-status-available-controller ok
[+]poststarthook/kube-apiserver-autoregistration ok
[+]autoregister-completion ok
[+]poststarthook/apiservice-openapi-controller ok
[+]poststarthook/apiservice-openapiv3-controller ok
[+]poststarthook/apiservice-discovery-controller ok
[+]shutdown ok
readyz check passed
```

>You can get the cluster info using the following command.

```console
root@hirmand:~# kubectl cluster-info
Kubernetes control plane is running at https://192.168.1.201:6443
CoreDNS is running at https://192.168.1.201:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

Also we can see what's running by `crio`
```console
root@hirmand:~# crictl ps
CONTAINER           IMAGE                                                              CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
9d2e49e04a8c6       cbb01a7bd410dc08ba382018ab909a674fb0e48687f0c00797ed5bc34fcc6bb4   27 minutes ago      Running             coredns                   0                   73c63dace340f       coredns-76f75df574-jx2tr
80b28eed7a261       cbb01a7bd410dc08ba382018ab909a674fb0e48687f0c00797ed5bc34fcc6bb4   27 minutes ago      Running             coredns                   0                   17b3229fcba67       coredns-76f75df574-cpthh
43aa435ebf9b9       cc8c46cf9d741d1e8a357e5899f298d2f4ac4d890a2d248026b57e130e91cd07   28 minutes ago      Running             kube-proxy                0                   142a0879d0aff       kube-proxy-v24nc
ff08c6467722c       9cffb486021b39220589cbd71b6537e6f9cafdede1eba315b4b0dc83e2f4fc8e   28 minutes ago      Running             kube-scheduler            0                   a3651339cefcb       kube-scheduler-hirmand
aa743dad754ae       32fe966e5c2b2a05d6b6a56a63a60e09d4c227ec1742d68f921c0b72e23537f8   28 minutes ago      Running             kube-controller-manager   0                   612119d9b2305       kube-controller-manager-hirmand
778f57020bf6a       a2e0d7fa8464a06b07519d78f53fef101bb1bcf716a85f2ac8b397f1a0025bea   28 minutes ago      Running             kube-apiserver            0                   b8239c12b0245       kube-apiserver-hirmand
47de3e407aa19       3861cfcd7c04ccac1f062788eca39487248527ef0c0cfd477a83d7691a75a899   28 minutes ago      Running             etcd                      0                   8fee4905f4136       etcd-hirmand
```

If you forget the command for joining another node as worker node, you can use the below command:
```
kubeadm token create --print-join-command

```

#### Setup Kubernetes Metrics Server

```
kubectl apply -f https://raw.githubusercontent.com/techiescamp/kubeadm-scripts/main/manifests/metrics-server.yaml

```

```
kubectl top nodes

```


---

## 2. Setup the Worker(s)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n25z7vpwle6rvlonjztw.png)

(Photo from the video)

Again, I followed the instruction [here](https://devopscube.com/setup-kubernetes-cluster-kubeadm/)








