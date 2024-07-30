## Day 26/40
# Kubernetes Network Policies Explained
[Video Link](https://www.youtube.com/watch?v=eVtnevr3Rao)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


And finally, `Network Policy`!
In this section we will discuss about `network policy` in `kubernetes` cluster. It's the last topic in `Security` section.

Let's quickly have a look at to the `network flow`.
- Sample 1 a simple one:
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ohar0230tzrm21jbiqr2.png)

(Photo from the video)

- Sample 2 in a `Kubernetes` cluster:
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wmpd5jw8b9qjhgroxtpd.png)

(Photo from the video)

We need `Network Policy` because it implements certain rules, certain restrictions and certain policies in our network, so each pod can or cannot communicate with other pod based on these policies which are enforce by the `CNI` plugins.
But not every `CNI` supports `Network Policy` for instance `flannel` and `kindnet`, as we are using `kind` for running our `kubernetes` cluster, they don't support these things :).
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/42i5z5go6ftb41ga581m.png)






>If you want to control traffic flow at the IP address or port level for TCP, UDP, and SCTP protocols, then you might consider using Kubernetes NetworkPolicies for particular applications in your cluster. NetworkPolicies are an application-centric construct which allow you to specify how a pod is allowed to communicate with various network "entities" (we use the word "entity" here to avoid overloading the more common terms such as "endpoints" and "services", which have specific Kubernetes connotations) over the network. NetworkPolicies apply to a connection with a pod on one or both ends, and are not relevant to other connections.
[source](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

















