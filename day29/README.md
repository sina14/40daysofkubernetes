## Day 29/40
# Kubernetes Volume Simplified | Persistent Volume, Persistent Volume Claim & Storage Class
[Video Link](https://www.youtube.com/watch?v=2NzYX8_lX_0)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)



This section is about `pv` persistent volume, `pvc` persistent volume claim, storage class and some other concepts about storage.

Let's assume a `pod` like the below, which has a `volume` named `redis-storage` and an `emptyDir` that is not persistent because we don't mount it somewhere.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
spec:
  containers:
  - image: redis
    name: redis
    volumeMounts:
      - name: redis-storage
        mountPath: /data/redis
  volumes:
    - name: redis-storage
      emptyDir: {}

```

```bash
root@localhost:~# kubectl apply -f day29-storage.yaml
pod/redis-pod created
root@localhost:~# kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
redis-pod   1/1     Running   0          10s
root@localhost:~# kubectl exec -it redis-pod -- bash
root@redis-pod:/data# mount | grep redis
/dev/vda1 on /data/redis type ext4 (rw,relatime,discard,errors=remount-ro)

```
If the `pod` killed or restarted, the data is still persist, but if the `pod` deleted, the data will be lost.










