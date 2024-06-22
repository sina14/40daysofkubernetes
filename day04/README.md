## Day 4/40
# Why Kubernetes Is Used - Kubernetes Simply Explained
[Video Link](https://www.youtube.com/watch?v=lXs1VCWqIH4)
@piyushsachdeva 

We're going to understand some fundamentals about Kubernetes.
Why, What and How!

Assume we have an application, and it contains some containers such as database, backend, frontend and ... and everyone is happy with them :)

If one of those containers failed, you cannot find a happy person in your team or clients because of a major incident and a downtime.

So we need to take care of them everytime and have some instance of them if the application needs help. I mean, if container A failed, there is at least one another container, let's say container B which can do exactly the duty of container A.

Also, we need something else to manage our workloads such as:

1. Container Networking
2. Resource Management
3. Security
4. High Availability
5. Fault Tolerance
6. Service Discovery
7. Simplified operations
8. Resilience
9. Scalability
10. Load Balancing
11. Orchestration
and so on.


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c7popw5abu99kx5vqqt7.png)

But we have to remember: "**Kubernetes is not always the solution!**"
Specially if we have small application with a few containers, it is waste of resources, money and administration workload on our Operation team. 
