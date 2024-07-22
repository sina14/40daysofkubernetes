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















