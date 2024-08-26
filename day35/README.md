## Day 35/40
# Kubernetes ETCD Backup And Restore Explained
[Video Link](https://www.youtube.com/watch?v=R2wuFCYgnm4)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In the section we're looking at `etcd` backup and restore which is a very important for `Kubernetes` administrators.
We need to take the backup of objects.

- Get all components in yaml format into yaml file:

```sh
root@localhost:~# kubectl get all -A -o yaml > backup.yaml

```

It's not efficient way to backing up the cluster because we didn't backup the persistent data and some other data which is not part of these manifests.

- `etcd` manifest's spec

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2os7nc8qmqh0fiiwlga7.png)

(Photo from the video)

- The `/var/lib/etcd` is the default directory which `etcd` data stores its configuration data.

- The `https://127.0.0.1:2379` is where `etcd` client is listening and the requests of `kubectl` sent by `api-server` to this url.

- The keys of `etcd` that is sent are mentioned too.
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --key-file=/etc/kubernetes/pki/etcd/server.key
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

- We have 2 volume mounts:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5im6iwq3f5pyjy5lfc7r.png)

(Photo from the video)

    - `etcd-data`: /var/lib/etcd - the data directory
    - `etcd-cets`: /etc/kubernetes/pki/etcd - the cert keys directory


- Install the `etcdctl` utility

```sh
root@localhost:~# apt install etcd-client -y

```

- Create an `env` for latest version of the utility because it uses version 2 by default.

```sh
root@localhost:~# export ETCDCTL_API=3

```

- Built-in snapshot
[source](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#built-in-snapshot)

```console
root@localhost:~# etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/etcd-backup

```













