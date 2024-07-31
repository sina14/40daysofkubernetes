## Day 27/40
# Setup a Multi Node Kubernetes Cluster Using Kubeadm
[Video Link](https://www.youtube.com/watch?v=WcdMC3Lj4tU)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


We're going to setup a multi-nodes `Kubernetes` cluster with [`kubeadm`](https://kubernetes.io/docs/reference/setup-tools/kubeadm/).
[Installation Guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

Before we begin, please take a look at the below flowchart and answer some questions to identifying the purpose of installation.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/77qh17fflz15clkf7xnj.png)
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

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/45phbopcu024cirj9394.png)
(Photo from the video)

After opening the neccessary ports and protocols, we have the below steps to continue:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tlv14o5f12k4obfgdqf5.png)
(Photo from the video)














