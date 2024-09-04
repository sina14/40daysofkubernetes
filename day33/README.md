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


**Note** the `Load-Balancing` companies such as `Nginx`, develops a `Kubernetes Ingress-Controller` that when it would be deployed through `Helm-Chart` or a manifest, read a `yaml` file.


**Note** the `Kubernetes Service` can provides load-balancing but with basic features such as Round robin load balancing. But we know there are lots of enterprise technologies that we might need.


**Note** we need to create a `ingress-resource` that `ingress-controller` will watch and create and configure `load-balancer` based on it.


- Some `ingress-controller` for `Kubernetes`:
> 1. Emissary
> 2. Nginx
> 3. HAProxy
> 4. Enovy
> 5. Traefik
> 6. F5 Container
> 7. Contour

[source](https://amazic.com/list-of-the-top-ingress-controllers-for-kubernetes/)


### Demo

#### 1. Clone files from repository

[Repository](https://github.com/piyushsachdeva/CKA-2024/tree/main/Resources/Day33/Flask)

```sh
root@sinaops:/opt/Flask# tree .
.
├── Dockerfile
├── app.py
├── k8s
│   ├── deployment.yaml
│   ├── ingress.yaml
│   └── service.yaml
└── requirements.txt

1 directory, 6 files
```

#### 2. Build the image and push to my repository

```sh
root@sinaops:/opt/Flask# docker build -t sinatavakkol/my-ingress-demo:v1 .

```

```sh
root@sinaops:/opt/Flask# docker images
REPOSITORY                                             TAG           IMAGE ID       CREATED          SIZE
sinatavakkol/my-ingress-demo                           v1            21794fe19661   19 seconds ago   137MB
...

```

```sh
root@sinaops:/opt/Flask# docker push sinatavakkol/my-ingress-demo:v1
The push refers to repository [docker.io/sinatavakkol/my-ingress-demo]
c15e83a40dc3: Pushed
e99cb889ac18: Pushed
34b40c4769c3: Pushed
d56439f47b97: Pushed
b8594deafbe5: Mounted from library/python
8a55150afecc: Mounted from library/python
ad34ffec41dd: Mounted from library/python
f19cb1e4112d: Mounted from library/python
d310e774110a: Mounted from library/python
v1: digest: sha256:e337d05667d7f0370268bff90dea054d49079a02f5be7196a9b74c237fd2e770 size: 2202

```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3hz6caatp0r72ynq5jwf.png)


#### 3. Replace the right image name for deployment file in `k8s` directory

```sh
root@sinaops:/opt/Flask# vim k8s/deployment.yaml

```
**Line 19**


#### 4. Apply the deployment and service manifests

```sh
root@sinaops:/opt/Flask# kubectl apply -f k8s/deployment.yaml
deployment.apps/hello-world created
root@sinaops:~# kubectl get deploy -o wide
NAME          READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS    IMAGES                            SELECTOR
hello-world   1/1     1            1           17m   hello-world   sinatavakkol/my-ingress-demo:v1   app=hello-world
root@sinaops:/opt/Flask# kubectl apply -f k8s/service.yaml
service/hello-world created
root@sinaops:/opt/Flask# kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
hello-world   ClusterIP   10.110.58.148   <none>        80/TCP    11s
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP   8d

```

- Test the service:

```sh
root@sinaops:/opt/Flask# curl 10.110.58.148
Hello, World!

```


#### 5. The ingress resource

Based on the rule on the resource, if someone who go for `example.com`, followed by `/`, he reached to `hello-world` app followed by `port: 80`.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: "example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-world
            port:
              number: 80

```

```sh
root@sinaops:/opt/Flask# kubectl apply -f  k8s/ingress.yaml
ingress.networking.k8s.io/hello-world created
root@sinaops:/opt/Flask# kubectl get ingress
NAME          CLASS   HOSTS         ADDRESS   PORTS   AGE
hello-world   nginx   example.com             80      25s

```

As we can see, there's not any address assigned because nobody is watching this resource!
So we need an `Ingress-Controller`.
Let's use **nginx ingress controller**.


#### 6. Ingress Controller

There are 2 types of **nginx ingress controller**, one is for `Kubernetes` community and one is for `Nginx` developer:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6fckdjnrdgpjo6yyi1tw.png)

**Ingress-NGINX Controller** vs **NGINX Ingress Controller**.

[source](https://kubernetes.github.io/ingress-nginx/deploy/#local-testing)

It continues on aws cloud ... I have no access to it


---


nginx contoller 


- Because the cloud controller manager doesn't exist in this `Kubernetes` cluster:

![image](https://github.com/user-attachments/assets/1195eb57-b1bf-47e0-b0b8-8f1ac90707f5)


- We edit `ingress-contoller` from `loadBalancer` type to `nodePort` so it will work:

![image](https://github.com/user-attachments/assets/6ba6f767-c0e2-45ee-9327-54225eccfd0a)

![image](https://github.com/user-attachments/assets/41f7d514-7509-4c0e-b3e1-f4564efacacd)


- Delete `ingress-resource` and create it again:

![image](https://github.com/user-attachments/assets/0f5423c8-e1bc-479b-99ce-632dc2a4a2fc)


![image](https://github.com/user-attachments/assets/1e497f0b-5507-467d-b242-bebf47601cfc)


![image](https://github.com/user-attachments/assets/cc90df41-2acd-4b1d-9302-3aadbb2f6bba)

- Because we told the `load-balancer` just route when domain name won't get `ip-address`, we should to edit /etc/hosts by `nodePort` IP address.

![image](https://github.com/user-attachments/assets/46c57567-1287-4d45-97da-adb2d5bfd982)

















