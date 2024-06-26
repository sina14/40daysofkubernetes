## Day 5/40
# What is Kubernetes
[Video Link](https://www.youtube.com/watch?v=SGGkUCctL4I)
@piyushsachdeva 

We're going to look at `Kubernetes` architecture in depth.
1. `Control Plane` components.
2. Why it's needed.
3. How do they work?

<img style="float: center;" src="https://github.com/sina14/40daysofkubernetes/blob/main/day05/kubernetes_arch.drawio.png">


---
- `Node` is nothing but a virtual machine.
- `Control Plane` is a virtual machine that hosts many administrative components, like board in companies!

- Actual works are done by `Worker Node`.
- `POD` is the smallest deployable unit that can be managed by `Kubernetes` which can include one or multiple containers.
<br/>
- `API Server` acts as the central management point of the entire `Kubernetes` cluster.
- `Schedular` is that it automatically decides how to distribute workloads across a cluster of servers.
<br/>
- `Controller Manager` which handles all interactions with `node controller`, `namespace controller`, `deployment controller`,  `replication controller` and so on, and make sure everything is up & running and monitored.
- `etcd` is a key-value datastore which stores the data required to manage clusters. Importantly.
<br/>
- `kubelet` functions as an agent within nodes and is responsible for the running of pod cycles within each node.
- `kube-proxy` controls traffic routing and network connectivity for services within the cluster.
<br/>
