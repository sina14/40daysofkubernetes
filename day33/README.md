## Day 33/40
# Kubernetes Ingress Tutorial | Ingress Explained by ‪[@AbhishekVeeramalla‬](https://www.youtube.com/@AbhishekVeeramalla)
[Video Link](https://www.youtube.com/watch?v=kf3UjITS91M)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)



In this part, we are going to learn about `Ingress` in `Kubernetes` cluster.

> In Kubernetes, Ingress is a resource type similar to Service, that allows you to easily route HTTP and HTTPS traffic entering the cluster through a single entry point to different services inside the cluster.
> Traffic routing is defined by rules specified on the Ingress resource.

[source](https://spacelift.io/blog/kubernetes-ingress#what-is-ingress-in-kubernetes)

> A Kubernetes cluster can have multiple associated Ingress controllers, and each one works similarly to a deployment controller.
> It is event-driven and gets triggered when, for example, a user creates, updates or deletes an Ingress.
> Once triggered, an Ingress controller tracks a cluster’s Ingress resources and reads the Ingress specifications.
> Then, it makes the configuration (in YAML or JSON) intelligible for the reverse proxy so it can provide the cluster with the resources it requires.

[source](https://www.solo.io/topics/kubernetes-api-gateway/kubernetes-ingress/)


> - Benefits of Kubernetes Ingress:
>    1. **Load Balancing**: Ingress can distribute traffic across multiple replicas of a service, ensuring high availability and scalability.
>    2. **SSL/TLS Termination**: Ingress controllers can handle SSL/TLS termination, offloading the encryption/decryption workload from the application services.
>    3. **Name-based Virtual Hosting**: Ingress allows multiple services to be exposed on the same IP address, using different hostnames.
>    4. **Path-based Routing**: Ingress can route traffic to different services based on the URL path, simplifying the management of complex applications.
>    5. **Canary Deployments**: Ingress can be used to implement canary deployments, gradually routing traffic to a new version of a service for testing purposes.

[source](https://konghq.com/blog/learning-center/what-is-kubernetes-ingress)

#### Why ingress?

Before having `ingress` service, we review some disadvantages of a `Load-Balancer` services:
    1. It's cloud-provider dependent.
    2. It's costly and expensive affair and approach.
    3. Security; we don't too much control of `Load-Balancer` service and we have to use everything the cloud provider decides.
So this is where `Ingress` comes to picture, when we need a `load-balancer`, an `API`, a `DDOS` protection, some `white-listed` addresses, a `WAF` and so on.


- There are some terms we need to understand:
>    1. Ingress resource - is a standard configuration object for an ingress controller.
>    2. Ingress controller - is a configurable proxy running in the cluster that is typically composed of a control plane and a data plane.
>    3. Load balancer - a cloud provider creates a load balancer in front of the nodes. Clients can access applications via the IP of the load balancer.

[source](https://blog.getambassador.io/getting-edgy-3-types-of-kubernetes-ingress-nodeports-load-balancers-and-ingress-controllers-b40ec8c0edb5)





---

2. What's ingress
  what it is doing?
   some kind of ing. controllers from some company

4. 19:00 hands-on demo

clone a repo
docker build image
push to registry

![image](https://github.com/user-attachments/assets/67514bc1-e68c-4e7e-8acf-01778fcd1a91)


create deployment
create service

![image](https://github.com/user-attachments/assets/3ef014b2-9d8c-4de3-834c-e301498b01d8)


curl svc_ip
cluster_ip or node_port

![image](https://github.com/user-attachments/assets/d3632662-ced9-421c-a337-38118f113bf6)


create ingress

2 types of nginx ingress controller

![image](https://github.com/user-attachments/assets/a029f9c0-6cbd-4824-9084-564c81811197)


nginx contoller 


because the cloud controller manager doesn't exist in this kuber cluster

![image](https://github.com/user-attachments/assets/1195eb57-b1bf-47e0-b0b8-8f1ac90707f5)


we edit ing contoller from load balancer to node port so it will work

![image](https://github.com/user-attachments/assets/6ba6f767-c0e2-45ee-9327-54225eccfd0a)

![image](https://github.com/user-attachments/assets/41f7d514-7509-4c0e-b3e1-f4564efacacd)


delete ing and create it again

![image](https://github.com/user-attachments/assets/0f5423c8-e1bc-479b-99ce-632dc2a4a2fc)


![image](https://github.com/user-attachments/assets/1e497f0b-5507-467d-b242-bebf47601cfc)


![image](https://github.com/user-attachments/assets/cc90df41-2acd-4b1d-9302-3aadbb2f6bba)

because we told the load balancer just route when domain name get not ip address
we should to edit /etc/hosts by nodeport ip address

![image](https://github.com/user-attachments/assets/46c57567-1287-4d45-97da-adb2d5bfd982)



Useful sources:

[source0](https://spacelift.io/blog/kubernetes-ingress)
[source1](https://www.solo.io/topics/kubernetes-api-gateway/kubernetes-ingress/)
[source2](https://kubernetes.io/docs/concepts/services-networking/ingress/)
[source3](https://konghq.com/blog/learning-center/what-is-kubernetes-ingress)
















