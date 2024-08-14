
# EDITING in Progress!

[source0](https://spacelift.io/blog/kubernetes-ingress)
[source1](https://www.solo.io/topics/kubernetes-api-gateway/kubernetes-ingress/)
[source2](https://kubernetes.io/docs/concepts/services-networking/ingress/)
[source3](https://konghq.com/blog/learning-center/what-is-kubernetes-ingress)

1. Why ingress
   disadvantages
   advantages
   
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












