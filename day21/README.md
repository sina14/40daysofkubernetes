## Day 21/40
# Manage TLS Certificates In a Kubernetes Cluster - Create Certificate Signing Request
[Video Link](https://www.youtube.com/watch?v=LvPA-z8Xg4s)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this part, we're looking at `tls` specific for the `kubernetes` clusters, how does it working and how we actually create and manage TLS `certificate`.

- The CA, Server certificate signing request, issuing the certification, clients have trusted certificate of the server signing by CA, client certificate, server certificate and root certificate in the below image:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/lruthimi87keiw2skov1.png)
 
(Photo from the video)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xwz1vwo99pcv2kof4u2e.png)

(Photo from the video)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sstrbkuz3vvi1yqfunyd.png)

(Photo from the video)

- As it's shown in the photos, we need certificates for user, client, server and all components in `kubernetes` cluster.

- So, if when we see `.crt` in a key extension, it's public key or certificate, and when the extension is `.key`, it's private key.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f8hnj8n4kcjkkg3nl8eb.png)

(Photo from the video)


### Demo

Let's say you are the administrator of a `kubernetes` cluster and you need to create user access for new admin user:

#### 1. Create private key
```console
root@localhost:~# openssl genrsa -out adam.key 2048
root@localhost:~# cat adam.key
-----BEGIN PRIVATE KEY-----
MIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQC3+5e3V+vG+yCT
...
obyArKL7NDYLDCYDL9u60YNmPw==
-----END PRIVATE KEY-----

```

#### 2. Create a CertificateSigningRequest using the key
```console
root@localhost:~# openssl req -new -key adam.key -out adam.csr -subj "/CN=adam"
root@localhost:~# cat adam.csr
-----BEGIN CERTIFICATE REQUEST-----
MIICVDCCATwCAQAwDzENMAsGA1UEAwwEYWRhbTCCASIwDQYJKoZIhvcNAQEBBQAD
...
frKCaPO/PvuScKlKT4khh7xI92uqPFrS
-----END CERTIFICATE REQUEST-----

```

#### 3. Create `yaml` file and encode `.csr` into `base64` and remove the **new lines** and then replace it into the below `yaml` file, `request` section.

```console
root@localhost:~# cat adam.csr | base64 | tr -d "\n"
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFV...1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==root@localhost:~#
```

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: adam
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFV...1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 864000  # 10 days
  usages:
  - client auth
```
#### 4. Apply the `yaml` file
```console
root@localhost:~# kubectl apply -f adam-csr.yaml
certificatesigningrequest.certificates.k8s.io/adam created
root@localhost:~# kubectl get csr
NAME   AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
adam   27s   kubernetes.io/kube-apiserver-client   kubernetes-admin   10d                 Pending

```

```console
root@localhost:~# kubectl describe csr adam
Name:         adam
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"adam"},"spec":{"expirationSeconds":864000,"request":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFV...1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}

CreationTimestamp:   Mon, 22 Jul 2024 17:37:09 +0000
Requesting User:     kubernetes-admin
Signer:              kubernetes.io/kube-apiserver-client
Requested Duration:  10d
Status:              Pending
Subject:
         Common Name:    adam
         Serial Number:
Events:  <none>
```

#### 5. It needs to be approved by `CA`, in our case an internal `CA`, because of the `CONDITION` is in `Pendig` state.

```console
root@localhost:~# kubectl certificate approve adam
certificatesigningrequest.certificates.k8s.io/adam approved
root@localhost:~# kubectl get csr
NAME   AGE    SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
adam   6m3s   kubernetes.io/kube-apiserver-client   kubernetes-admin   10d                 Approved,Issued

```

#### 5. Share the certificate with user

```console
root@localhost:~# kubectl get csr adam -o yaml > adam-issued-cert.yaml
root@localhost:~# cat adam-issued-cert.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"adam"},"spec":{"expirationSeconds":864000,"request":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRV...lMKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}
  creationTimestamp: "2024-07-22T17:37:09Z"
  name: adam
  resourceVersion: "2740320"
  uid: c0fd62e2-f62a-4742-a7b6-0e0ce9a6e5a7
spec:
  expirationSeconds: 864000
  groups:
  - kubeadm:cluster-admins
  - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6R...QTy9QdnVTY0tsS1Q0a2hoN3hJOTJ1cVBGclMKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
  username: kubernetes-admin
status:
  certificate: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM5RENDQWR5Z0F3SUJBZ0lRVzFSbUg1d...PWlN6d2sxMTJWNTlGa1MwZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
  conditions:
  - lastTransitionTime: "2024-07-22T17:43:02Z"
    lastUpdateTime: "2024-07-22T17:43:02Z"
    message: This CSR was approved by kubectl certificate approve.
    reason: KubectlApprove
    status: "True"
    type: Approved
```

#### 6. Decode the certificate 
```sh
root@localhost:~# echo "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM5RENDQWR5Z0F3SUJBZ0lRVzFSbUg1...lPWlN6d2sxMTJWNTlGa1MwZz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K" | base64 -d
-----BEGIN CERTIFICATE-----
MIIC9DCCAdygAwIBAgIQW1RmH5uzz5wY6wI4k6tVMDANBgkqhkiG9w0BAQsFADAV
...
mS5uQo/mZY31y6AFDMpcRsUmgJQye5eKmBu/YOZSzwk112V59FkS0g==
-----END CERTIFICATE-----

```
[source](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user)
[Another demo](https://yuminlee2.medium.com/kubernetes-generate-certificates-for-normal-users-using-certificates-api-7ba71170aa52)










