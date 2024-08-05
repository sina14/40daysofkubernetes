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

---

At first we need some definitions to have better understanding the concepts.
[source](https://blog.mayadata.io/understanding-persistent-volumes-and-pvcs-in-kubernetes)
>- **Volume**: In `Kubernetes`, `volume` abstractions are used to provide an API that abstracts the physical implementation of `storage` from how it is consumed by application resources.
>Kubernetes supports two major types of Volumes:

    - Ephemeral Volumes - These are used for applications that need storage but do not need to access the data after a restart:

          - emptyDir
          - Secrets, ConfigMaps and the downwardAPI
          - CSI (Container Storage Interface) Ephemeral Volumes
          - Generic Ephemeral Volumes
    
    - Persistent Volumes - This is an API object that represents an abstract implementation of physical storage to be used by PODs, but they last beyond a POD’s lifetime.

> - **Persistent Storage**: Once a CSI plugin has been set up and is running in Kubernetes, resources and users can consume volumes using the Kubernetes Storage API objects: `Persistent Volumes`, `Persistent Volume Claims` and `Storage Classes`.

    - **Persistent Volumes (PV)**: is a piece of `storage` available to the `cluster`. The `PV` exposes object, file and block storage systems by capturing the details of its implementation protocol, be it `iSCSI` (SCSI over Internet), `NFS` or any storage systems offered by specific vendors and cloud providers. The `Persistent Volume` has its lifecycle independent of any `POD` consuming it.
    - **Persistent Volume Claims (PVC)**: When a user requests for `PV` storage, they use a `Persistent Volume Claim` as a `Kubernetes` object that requests specific storage requirements such as `access modes` and size.

> - **The Lifecycle of `PV`s and `PVC`s**:
  1. Provisioning
  2. Binding
  3. Using
  4. Reclaiming

> - **Reclaim Policy**:
    1. Retain
    2. Delete
    3. Recycle

> Some of the most popular CSI Plugins for Kubernetes include:
    - AWS Elastic Block Storage
    - Azure disk
    - BeeGFS
    - CephFS
    - Dell EMC PowerMax
    - GCE Persistent Disk
    - Google Cloud Filestore
    - GlusterFS
    - Huawei Storage CSI
    - HyperV CSI
    - IBM Block Storage
    - OpenEBS
    - Portworx
    - Pure Storage CSI
The complete list is [here](https://kubernetes-csi.github.io/docs/drivers.html)


> - **Access Mode**:
    - ReadWriteOnce - the volume can be mounted as read-write by a single node. 
    - ReadOnlyMany - the volume can be mounted as read-only by many nodes.
    - ReadWriteMany - the volume can be mounted as read-write by many nodes.
    - ReadWriteOncePod - the volume can be mounted as read-write by a single Pod.
[source](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)


---

![pv_and_pvc](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kkkzr3r3qv8tjh959crs.png)

(Photo from the video)

#### Demo
based on the diagram

#### 1. Create a PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  storageClassName: standard
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/home/day29"
```

```bash
root@localhost:~# kubectl apply -f day29-pv.yaml
persistentvolume/task-pv-volume created
root@localhost:~# kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
task-pv-volume   1Gi        RWO            Retain           Available                          <unset>                          10s
```

#### 1. Create a PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi

```

```bash
root@localhost:~# kubectl apply -f day29-pvc1.yaml
persistentvolumeclaim/task-pv-claim created
root@localhost:~# kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
task-pv-volume   1Gi        RWO            Retain           Available                          <unset>                          4m4s
root@localhost:~# kubectl get pvc
NAME            STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
task-pv-claim   Pending                                      standard       <unset>                 19s

```
It remains in `Pending` status until a `pod` would claim it.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: task-pv-pod
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: task-pv-claim
  containers:
    - name: task-pv-container
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
```

```bash
root@localhost:~# kubectl get pod
NAME          READY   STATUS    RESTARTS   AGE
task-pv-pod   1/1     Running   0          9s
root@localhost:~# kubectl get pv
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
task-pv-volume   1Gi        RWO            Retain           Bound    default/task-pv-claim   standard       <unset>                          32s
root@localhost:~# kubectl get pvc
NAME            STATUS   VOLUME           CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
task-pv-claim   Bound    task-pv-volume   1Gi        RWO            standard       <unset>                 24s
```















