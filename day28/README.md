## Day 28/40
# Docker Volume Explained - Docker Bind Mount - Docker Persistent Storage
[Video Link](https://www.youtube.com/watch?v=ZAPX21TMkkQ)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)



In this section, we're looking at `docker storage`, how it works behind the scene and how we can use storage in `docker`.

Assume we need to add a writable layer, `container layer` to our application, which is persist even it would be down or restarted after creation.

The `container layer` nothing but the writable copy of `Read-only layer` but it's not persistent.
So with the help of `volume` we can have persistent data for a container.

`Storage drivers` is responsible for writing data in `container layer`. It also stores layers within `container`

`Volume drivers` is responsible to make the data persistent.

[Read more](https://docs.docker.com/storage/storagedriver/#storage-drivers-versus-docker-volumes), [Docker Volume](https://docs.docker.com/storage/volumes/)

> The Docker Engine provides the following storage drivers on Linux:

| Driver | Description |
| ------ | ------ |
| `overlay2` | `overlay2` is the preferred storage driver for all currently supported Linux distributions, and requires no extra configuration. |
| `fuse-overlayfs` | `fuse-overlayfsis` preferred only for running Rootless Docker on an old host that does not provide support for rootless `overlay2`. The `fuse-overlayfs` driver does not need to be used since Linux kernel 5.11, and `overlay2` works even in rootless mode. Refer to the [rootless mode documentation](https://docs.docker.com/engine/security/rootless/) for details. |
| `btrfs` and `zfs` | The `btrfs` and `zfs` storage drivers allow for advanced options, such as creating "snapshots", but require more maintenance and setup. Each of these relies on the backing filesystem being configured correctly. |
| `vfs` | The `vfs` storage driver is intended for testing purposes, and for situations where no copy-on-write filesystem can be used. Performance of this storage driver is poor, and is not generally recommended for production use. |

[source](https://docs.docker.com/storage/storagedriver/select-storage-driver/)

#### Creating a volume in docker

```console
root@localhost:~# docker volume create data_vol
data_vol
root@localhost:~# docker volumes
docker: 'volumes' is not a docker command.
See 'docker --help'
root@localhost:~# docker volume ls
DRIVER    VOLUME NAME
local     data_vol
root@localhost:~# ls -lh /var/lib/docker/volumes/data_vol/
total 4.0K
drwxr-xr-x 2 root root 4.0K Aug  1 15:08 _data
root@localhost:~# ls -lh /var/lib/docker/volumes/data_vol/_data/
total 0
```

#### Mount new volume to a container

```console
root@sinaops:~# docker run -v data_vol:/app -dp 3000:3000 --name todo day02-todo
4c4a22dfd59e4b956eadc464ac93a64db1bfcbb3c23580e55a46b24ea8431091
root@sinaops:~# ls -lh /var/lib/docker/volumes/data_vol/_data/
total 164K
-rw-r--r--   1 root root  269 Aug  1 14:37 README.md
drwxr-xr-x 167 root root 4.0K Aug  1 15:23 node_modules
-rw-r--r--   1 root root  648 Aug  1 14:37 package.json
drwxr-xr-x   4 root root 4.0K Aug  1 15:23 spec
drwxr-xr-x   5 root root 4.0K Aug  1 15:23 src
-rw-r--r--   1 root root 144K Aug  1 14:37 yarn.lock
root@sinaops:~# du -s /var/lib/docker/volumes/data_vol/_data/
28196   /var/lib/docker/volumes/data_vol/_data/

```
After restarting the container, the data will be persistent.



