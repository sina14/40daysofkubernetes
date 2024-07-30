## Day 26/40
# Kubernetes Network Policies Explained
[Video Link](https://www.youtube.com/watch?v=eVtnevr3Rao)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


And finally, `Network Policy`!
In this section we will discuss about `network policy` in `kubernetes` cluster. It's the last topic in `Security` section.

Let's quickly have a look at to the `network flow`.


>If you want to control traffic flow at the IP address or port level for TCP, UDP, and SCTP protocols, then you might consider using Kubernetes NetworkPolicies for particular applications in your cluster. NetworkPolicies are an application-centric construct which allow you to specify how a pod is allowed to communicate with various network "entities" (we use the word "entity" here to avoid overloading the more common terms such as "endpoints" and "services", which have specific Kubernetes connotations) over the network. NetworkPolicies apply to a connection with a pod on one or both ends, and are not relevant to other connections.
[source](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

















