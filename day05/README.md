## Day 5/40
# What is Kubernetes
[Video Link](https://www.youtube.com/watch?v=SGGkUCctL4I)
@piyushsachdeva 

We're going to look at `Kubernetes` architecture in depth.
1. `Control Plane` components.
2. Why it's needed.
3. How do they work?


![Kubernetes_arch](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hrumznwa567ftsmdmlo6.png)

---
- `Node` is nothing but a virtual machine.
- `Control Plane` is a virtual machine that hosts many administrative components, like board in companies!

- Actual works are done by `Worker Node`.
- `POD` is the smallest deployable unit that can be managed by `Kubernetes` which can include one or multiple containers.

- `API Server` acts as the central management point of the entire `Kubernetes` cluster.
- `Schedular` is that it automatically decides how to distribute workloads across a cluster of servers.

- `Controller Manager` which handles all interactions with `node controller`, `namespace controller`, `deployment controller`,  `replication controller` and so on, and make sure everything is up & running and monitored.
- `etcd` is a key-value datastore which stores the data required to manage clusters. Importantly.

- `kubelet` functions as an agent within nodes and is responsible for the running of pod cycles within each node.
- `kube-proxy` controls traffic routing and network connectivity for services within the cluster.

- `kubectl` is a command-line tool which command communicates with the `kube-apiserver` and sends orders to the `control plane` node.

---
### Work Flow
1. `kubectl` to `api-server`: Please create the `pod`
2. `api-server` validate and authenticate the user with `etcd` and notify the `controller-manager`.
3. `controller-manager` tells the `api-server` it's ok to have a `POD`
4. `api-server` write the request in `etcd` database.
5. `etcd` says to `api-server` yes it's done.
6. `api-server` says to `scheduler` to watch the new request.
7. `schedular` monitors and find out there's a `pod` creation request.
8. `schedular` says to `api-server` that I found a very special node for deploying the new `pod`.
9. `api-server` interacts with the `kubelet` of the selection `node`.
10. `kubelet` asks `docker` (or other alternative) to build the `container` with requeted `image`.
11. `docker` says to `'kubelet`, hey deploy what I built.
12. `kubelet` run the `pod` and sends back the detail to `api-server` that the `pod` is deployed.
13. `api-server` update the related record in `etcd` database.
14. `api-server` sends the detail to `kubectl` and the `user`.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/53okd63k478f28j2h2eu.png)

