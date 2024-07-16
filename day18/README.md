## Day 18/40
# Kubernetes Probes Explained | Liveness vs Readiness vs Startup Probes
[Video Link](https://www.youtube.com/watch?v=x2e6pIBLKzw)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


It's time to explain the **Health Probes** like:
- Liveness Probe
- Readiness Probe
- Startup Probe
and so on.
<p>
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ijmncxbp9ohf6revodjk.png)
<br/>(Photo from the video)
</p>
---
In Kubernetes, a probe is a mechanism used to determine the health and readiness of a container or application running within a pod. Probes are defined in the pod specification and are performed periodically to ensure the proper functioning of the application.
- **Liveness Probe**: A liveness probe determines if a container is still running and functioning correctly.
- **Readiness Probe**: A readiness probe determines if a container is ready to receive incoming network traffic and serve requests.
- **Startup Probe**: A startup probe is used to determine if a container's application has started successfully. 

[source](https://kubeops.net/blog/kubernetes-probes)

















