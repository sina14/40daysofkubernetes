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

#### 1. Disable the `swap`

```console
root@srv001:~# swapoff -a
root@srv001:~# #sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
root@srv001:~# cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-ANF6LghHr6wZpLRjos2sOTAmBsMksbsGR8JcLny6mZwIJfIBBME4KIANwL3nQD9r / ext4 defaults 0 1
# /boot was on /dev/sda2 during curtin installation
```

#### 2. Forwaring IPv4 and letting iptables see bridged traffic

```console
root@srv001:~# sudo modprobe overlay
root@srv001:~# sudo modprobe br_netfilter
root@srv001:~# cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

```

#### 3. Apply `sysctl` parameters without reboot

```console
root@srv001:~# sudo sysctl --system
* Applying /etc/sysctl.d/10-console-messages.conf ...
kernel.printk = 4 4 1 7
* Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
* Applying /etc/sysctl.d/10-kernel-hardening.conf ...
kernel.kptr_restrict = 1
* Applying /etc/sysctl.d/10-magic-sysrq.conf ...
...
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
* Applying /etc/sysctl.conf ...

```

#### 4. Verify that the br_netfilter, overlay modules are loaded and check the other options

```console
root@srv001:~# lsmod | grep br_netfilter
br_netfilter           32768  0
bridge                307200  1 br_netfilter
root@srv001:~# lsmod | grep overlay
overlay               151552  0
root@srv001:~# sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1

```

#### 5. Install container runtime

```console
root@srv001:~# curl -LO https://github.com/containerd/containerd/releases/download/v1.7.14/containerd-1.7.14-linux-amd64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 45.5M  100 45.5M    0     0  1242k      0  0:00:37  0:00:37 --:--:-- 1353k
root@srv001:~# ls
containerd-1.7.14-linux-amd64.tar.gz  snap
root@srv001:~# tar Cxzvf /usr/local containerd-1.7.14-linux-amd64.tar.gz
bin/
bin/containerd-shim
bin/ctr
bin/containerd-shim-runc-v1
bin/containerd
bin/containerd-stress
bin/containerd-shim-runc-v2
root@srv001:~# curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1251  100  1251    0     0   2679      0 --:--:-- --:--:-- --:--:--  2684
root@srv001:~# ls
containerd-1.7.14-linux-amd64.tar.gz  containerd.service  snap  test_project.bash
root@srv001:~# mkdir -p /usr/local/lib/systemd/system/
root@srv001:~# mv containerd.service /usr/local/lib/systemd/system/
root@srv001:~# mkdir -p /etc/containerd
root@srv001:~# containerd config default | sudo tee /etc/containerd/config.toml
disabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
...
[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.metrics.shimstats" = "2s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[ttrpc]
  address = ""
  gid = 0
  uid = 0
root@srv001:~# sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
root@srv001:~# systemctl daemon-reload
root@srv001:~# systemctl enable --now containerd
Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /usr/local/lib/systemd/system/containerd.service.
root@srv001:~# systemctl status containerd
● containerd.service - containerd container runtime
     Loaded: loaded (/usr/local/lib/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2024-07-31 17:09:22 UTC; 7s ago
       Docs: https://containerd.io
    Process: 60923 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 60924 (containerd)
      Tasks: 7
     Memory: 13.5M
        CPU: 248ms
     CGroup: /system.slice/containerd.service
             └─60924 /usr/local/bin/containerd

Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.477949240Z" level=error msg="failed to load cni during init, please check C>
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.478275229Z" level=info msg=serving... address=/run/containerd/containerd.so>
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.478326187Z" level=info msg=serving... address=/run/containerd/containerd.so>
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.478418572Z" level=info msg="Start subscribing containerd event"
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.478457241Z" level=info msg="Start recovering state"
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.478537176Z" level=info msg="Start event monitor"
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.478565633Z" level=info msg="Start snapshots syncer"
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.478575184Z" level=info msg="Start cni network conf syncer for default"
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.478583511Z" level=info msg="Start streaming server"
Jul 31 17:09:22 srv001 containerd[60924]: time="2024-07-31T17:09:22.481571822Z" level=info msg="containerd successfully booted in 0.090792s"


```

#### 6. Install runc

```console
root@srv001:~# curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.12/runc.amd64
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 10.2M  100 10.2M    0     0  1114k      0  0:00:09  0:00:09 --:--:-- 1331k
root@srv001:~# sudo install -m 755 runc.amd64 /usr/local/sbin/runc

```

#### 7. Install CNI plugin

```console
root@srv001:~# curl -LO https://github.com/containernetworking/plugins/releases/download/v1.5.0/cni-plugins-linux-amd64-v1.5.0.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 45.9M  100 45.9M    0     0  1235k      0  0:00:38  0:00:38 --:--:-- 1317k
root@srv001:~# mkdir -p /opt/cni/bin
root@srv001:~# tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.0.tgz
./
./dhcp
./loopback
./README.md
./bandwidth
./ipvlan
./vlan
./static
./host-device
./LICENSE
./bridge
./dummy
./tuning
./vrf
./tap
./portmap
./firewall
./ptp
./host-local
./macvlan
./sbr

```


#### 8. Install kubeadm, kubelet and kubectl

```console
root@srv001:~# apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Hit:3 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Get:4 http://archive.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Get:5 http://archive.ubuntu.com/ubuntu jammy-security/main amd64 Packages [1,681 kB]
Fetched 1,938 kB in 4s (441 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
4 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@srv001:~# apt install -y apt-transport-https ca-certificates curl gpg
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
ca-certificates is already the newest version (20230311ubuntu0.22.04.1).
ca-certificates set to manually installed.
curl is already the newest version (7.81.0-1ubuntu1.16).
gpg is already the newest version (2.2.27-3ubuntu2.1).
gpg set to manually installed.
...
Scanning processes...
Scanning candidates...
Scanning linux images...

Restarting services...
Service restarts being deferred:
...
No VM guests are running outdated hypervisor (qemu) binaries on this host.
root@srv001:~# curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
root@srv001:~# echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /
root@srv001:~# apt update
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Hit:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease
Hit:3 http://archive.ubuntu.com/ubuntu jammy-backports InRelease
Hit:4 http://archive.ubuntu.com/ubuntu jammy-security InRelease
Get:5 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.29/deb  InRelease [1,189 B]
Get:6 https://prod-cdn.packages.k8s.io/repositories/isv:/kubernetes:/core:/stable:/v1.29/deb  Packages [11.5 kB]
Fetched 12.7 kB in 4s (3,522 B/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
4 packages can be upgraded. Run 'apt list --upgradable' to see them.
root@srv001:~# apt install -y kubelet=1.29.6-1.1 kubeadm=1.29.6-1.1 kubectl=1.29.6-1.1 --allow-downgrades --allow-change-held-packages
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
...

root@srv001:~# apt-mark hold kubelet kubeadm kubectl
kubelet set on hold.
kubeadm set on hold.
kubectl set on hold.
root@srv001:~# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"29", GitVersion:"v1.29.6", GitCommit:"062798d53d83265b9e05f14d85198f74362adaca", GitTreeState:"clean", BuildDate:"2024-06-11T20:22:13Z", GoVersion:"go1.21.11", Compiler:"gc", Platform:"linux/amd64"}
root@srv001:~# kubelet --version
Kubernetes v1.29.6
root@srv001:~# kubectl version --client
Client Version: v1.29.6
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3

```


#### 9. Configure `crictl` to work with `containerd`

```console
root@srv001:~# crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```


#### 10. Initialize `control-plane`

```console

```


#### 11. Prepare kubeconfig

```console

```


#### 12. Install calico

```console

```


#### 13. Install container runtime

```console

```


#### 14. Install container runtime

```console

```


#### 15. Install container runtime

```console

```


#### 16. Install container runtime

```console

```


#### 5. Install container runtime

```console

```


#### 5. Install container runtime

```console

```


#### 5. Install container runtime

```console

```


#### 5. Install container runtime

```console

```


#### 5. Install container runtime

```console

```








