## Day 20/40
# SSL/TLS Explained Simply
[Video Link](https://www.youtube.com/watch?v=njT5ECuwCTo)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


This is prerequisite of the next session which will be about `certification` in `Kubernetes`.
In this topic we will learn how `ssl` and `tls` work.

There are two types of key encryption:
1. Symmetric Encryption:
Which both side, client and server, use one key to encrypt and decrypt the traffic between them.
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sw04om8cghs954e98km9.png)
<br/>
(Photo from the video)

2. Asymmetric Encryption:
There are two key, public and private, that client encrypt the traffic with public key, and the private key which the server only has it, can decrypt the traffic.
 
- CA is certificate authority to validate if the public key is generated for the right server or domain name, and it can help us to distinguish between the clean connection between client and server and the traffic when man-in-the-middle is behind!
<br/>
For example:
<br/>
- Website Identity from our browser:<br/>

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xbvbxui0kkjaxf3af68y.png)

- The CA:<br/>

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/df01cw326celszpoh5rj.png)



