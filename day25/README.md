## Day 25/40
# Kubernetes Service Account - RBAC Continued
[Video Link](https://www.youtube.com/watch?v=k2iCq7IlMKM)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section, we are looking at `service account` 

#### What are service accounts?
>A `service account` is a type of non-human account that, in `Kubernetes`, provides a distinct identity in a `Kubernetes` cluster. Application Pods, system components, and entities inside and outside the `cluster` can use a specific `ServiceAccount`'s credentials to identify as that `ServiceAccount`. This identity is useful in various situations, including authenticating to the API server or implementing identity-based security policies.
[source](https://kubernetes.io/docs/concepts/security/service-accounts/#what-are-service-accounts)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nqqegd01m8jzip6u86xp.png)

(Photo from the video)

>When you create a cluster, `Kubernetes` automatically creates a `ServiceAccount` object named `default` for every namespace in your cluster.
[source](https://kubernetes.io/docs/concepts/security/service-accounts/#default-service-accounts)

```console
root@localhost:~# kubectl get sa
NAME      SECRETS   AGE
default   0         27d
root@localhost:~# kubectl describe sa default
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
root@localhost:~# kubectl get sa default -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2024-07-01T16:17:24Z"
  name: default
  namespace: default
  resourceVersion: "392"
  uid: 4c64b284-e8c3-4e70-a67e-cb7c0d5a379e

```
 
#### Create a service account

```console
root@localhost:~# kubectl create sa build-sa
serviceaccount/build-sa created
root@localhost:~# kubectl describe sa build-sa
Name:                build-sa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>

```

#### Manually create a long-lived API token for a ServiceAccount
>If you want to obtain an API token for a ServiceAccount, you create a new Secret with a special annotation, `kubernetes.io/service-account.name`.
[source](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

- Service account token
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: build-robot-secret
  annotations:
    kubernetes.io/service-account.name: build-sa
type: kubernetes.io/service-account-token
```

```console
root@localhost:~# kubectl apply -f day25-secret.yaml
secret/build-robot-secret created
root@localhost:~# kubectl get secret
NAME                 TYPE                                  DATA   AGE
build-robot-secret   kubernetes.io/service-account-token   3      8s
root@localhost:~# kubectl describe secret build-robot-secret
Name:         build-robot-secret
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: build-sa
              kubernetes.io/service-account.uid: 2f2bbf57-41ad-4be1-a4b6-618a093edd45

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImEwemh... R4gFq5INlcdOrbF-6yQEe9fz6n2znYoSmX3Qi-BKX3HL8dMbQ2McvXXTNbcr9T8Cnw3Sa2uJA2uoD8QKmzBKjzSSeac8ymUvq0kYgbIIC4ITdtZCA26hD54Hds3i92uoQ245Vfh9miW_YVHtkVgL9tCjrKJRfkEYEfd2H_Eijq-W6HPePUC7m1lvIviYZr1IcCfUDY8jHt8XwIVPs6JwzQnkirRWq-3bylmvNNR1W7FqwwADjv581mmwHSY4KoDpjM0T_a-kJCN8ufLI_m6o12Tw
ca.crt:     1107 bytes
namespace:  7 bytes

```



