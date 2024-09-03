## Day 40/40
# JSONPath Tutorial - Advanced Kubectl Commands
[Video Link](https://www.youtube.com/watch?v=l9_UDSaiFj4)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section we will deep dive into JSONPath from the beginners perspective and see how you can write advanced kubectl commands using JSONPATH


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gzmnqbik5uvz5calvy3o.png)


- Return result in `json` format by `api-server`

```
kubectl get nodes -o json

```

- Return result in `yaml` format by `api-server`

```
kubectl get nodes -o yaml

```

- Sample of jsonpath:

```sh
root@sinaops:~# kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}{"\n"}'
Ubuntu 24.04.1 LTS Ubuntu 22.04.2 LTS Ubuntu 22.04.4 LTS

```

- With custom column:

```sh
root@sinaops:~# kubectl get nodes -o='custom-columns=OsType:{.status.nodeInfo.osImage},KubeletVersion:{.status.nodeInfo.kubeletVersion}'
OsType               KubeletVersion
Ubuntu 24.04.1 LTS   v1.30.4
Ubuntu 22.04.2 LTS   v1.30.0
Ubuntu 22.04.4 LTS   v1.30.4

```

- With custom column and statement:

```sh
root@sinaops:~# kubectl get nodes -o=custom-columns='Host:{.status.addresses[?(@.type=="Hostname")].address},OsType:{.status.nodeInfo.osImage}'
Host         OsType
cloudy.net   Ubuntu 24.04.1 LTS
jolly-net    Ubuntu 22.04.2 LTS
sinaops      Ubuntu 22.04.4 LTS

```

- Sort by an item:

```sh
root@sinaops:~# kubectl get nodes
NAME         STATUS                     ROLES           AGE     VERSION
cloudy.net   Ready                      worker          6d      v1.30.4
jolly-net    Ready,SchedulingDisabled   worker          7d22h   v1.30.0
sinaops      Ready                      control-plane   8d      v1.30.4
root@sinaops:~# kubectl get nodes --sort-by=.status.nodeInfo.kubeletVersion
NAME         STATUS                     ROLES           AGE     VERSION
jolly-net    Ready,SchedulingDisabled   worker          7d22h   v1.30.0
cloudy.net   Ready                      worker          6d      v1.30.4
sinaops      Ready                      control-plane   8d      v1.30.4

```
