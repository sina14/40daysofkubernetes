## Day 40/40
# JSONPath Tutorial - Advanced Kubectl Commands
[Video Link](https://www.youtube.com/watch?v=l9_UDSaiFj4)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section we will deep dive into JSONPath from the beginners perspective and see how you can write advanced kubectl commands using JSONPATH


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gzmnqbik5uvz5calvy3o.png)


- Return result in `json` format by `api-server`

```
kubectl get nodes -o json

```

- Return result in `yaml` format by `api-server`

```
kubectl get nodes -o yaml

```


 ```sh
root@sinaops:~# kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}{"\n"}'
Ubuntu 24.04.1 LTS Ubuntu 22.04.2 LTS Ubuntu 22.04.4 LTS

```







