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

## Setup the Control plane(s)

**Note** I had some problem with the instruction that's introduced in the video and I followed another one [here](https://devopscube.com/setup-kubernetes-cluster-kubeadm/)

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

kubeadm join 192.168.1.201:6443 --token qj65ev.lndore1291y5nwnd \
        --discovery-token-ca-cert-hash sha256:c20e7329887dd9f2a66c36ac96ecb695d62e4ae9a4d90a9e00745af75bb727cb
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




