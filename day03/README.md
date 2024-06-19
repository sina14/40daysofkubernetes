## Day 3/40
# Multi Stage Docker Build - Docker Tutorial For Beginners
[Video Link](https://www.youtube.com/watch?v=ajetvJmBvFo)
@piyushsachdeva 

We're going to reduce the volume of our image which is built in Day 2, also for optimizing and improving the performance with multi-staging technique.

- Clone a repository include a simple app
```console
[node1] (local) root@192.168.0.8 ~
$ cd /opt ; git clone https://github.com/piyushsachdeva/todoapp-docker
Cloning into 'todoapp-docker'...
remote: Enumerating objects: 81, done.
remote: Counting objects: 100% (81/81), done.
remote: Compressing objects: 100% (45/45), done.
remote: Total 81 (delta 29), reused 73 (delta 26), pack-reused 0
Receiving objects: 100% (81/81), 186.07 KiB | 6.00 MiB/s, done.
Resolving deltas: 100% (29/29), done.
[node1] (local) root@192.168.0.8 /opt
$ cd todoapp-docker/
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ ls
README.md          package-lock.json  package.json       public             src
```

- Create a Dockerfile
```
FROM node:18-alpine AS installer

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM nginx:latest AS deployer

COPY --from=installer /app/build /usr/share/nginx/html
```

- Let's build image from Dockerfile
```console
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ docker build -t multi-stage .
[+] Building 85.0s (14/14) FINISHED                                                                                                                            docker:default
 => [internal] load build definition from Dockerfile                                                                                                                     0.0s
 => => transferring dockerfile: 243B                                                                                                                                     0.0s
 => [internal] load .dockerignore                                                                                                                                        0.0s
 => => transferring context: 2B                                                                                                                                          0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                                                                                          0.4s
 => [internal] load metadata for docker.io/library/node:18-alpine                                                                                                        0.4s
 => [installer 1/6] FROM docker.io/library/node:18-alpine@sha256:6937be95129321422103452e2883021cc4a96b63c32d7947187fcb25df84fc3f                                        5.8s
 => => resolve docker.io/library/node:18-alpine@sha256:6937be95129321422103452e2883021cc4a96b63c32d7947187fcb25df84fc3f                                                  0.0s
 => => sha256:05412f5b9ed819c373a2535804e473a155fc91bfb7adf469ec2312e056a9e87f 1.16kB / 1.16kB                                                                           0.0s
 => => sha256:e7d39d4d8569a6203be5b7a118d4d92526b267087023a49ee0868f7c50190191 7.23kB / 7.23kB                                                                           0.0s
 => => sha256:d25f557d7f31bf7acfac935859b5153da41d13c41f2b468d16f729a5b883634f 3.62MB / 3.62MB                                                                           0.2s
 => => sha256:f6124930634921d33d69a1a8b5848cb40d0b269e79b4c37c236cb5e4d61a2710 39.83MB / 39.83MB                                                                         0.9s
 => => sha256:22a81a0f8d1c30ce5a5da3579a84ab4c22fd2f14cb33863c1a752da6f056dc18 1.38MB / 1.38MB                                                                           0.2s
 => => sha256:6937be95129321422103452e2883021cc4a96b63c32d7947187fcb25df84fc3f 1.43kB / 1.43kB                                                                           0.0s
 => => extracting sha256:d25f557d7f31bf7acfac935859b5153da41d13c41f2b468d16f729a5b883634f                                                                                0.6s
 => => sha256:bd06542006fda4279cb2edd761a84311c1fdbb90554e9feaaf078a3674845742 447B / 447B                                                                               0.2s
 => => extracting sha256:f6124930634921d33d69a1a8b5848cb40d0b269e79b4c37c236cb5e4d61a2710                                                                                4.2s
 => => extracting sha256:22a81a0f8d1c30ce5a5da3579a84ab4c22fd2f14cb33863c1a752da6f056dc18                                                                                0.2s
 => => extracting sha256:bd06542006fda4279cb2edd761a84311c1fdbb90554e9feaaf078a3674845742                                                                                0.0s
 => [deployer 1/2] FROM docker.io/library/nginx:latest@sha256:56b388b0d79c738f4cf51bbaf184a14fab19337f4819ceb2cae7d94100262de8                                           9.1s
 => => resolve docker.io/library/nginx:latest@sha256:56b388b0d79c738f4cf51bbaf184a14fab19337f4819ceb2cae7d94100262de8                                                    0.0s
 => => sha256:56b388b0d79c738f4cf51bbaf184a14fab19337f4819ceb2cae7d94100262de8 10.27kB / 10.27kB                                                                         0.0s
 => => sha256:dca6c1f16ab4ac041e55a10ad840e6609a953e1b2ee1ec3e4d3dfe2b4dfbbf34 2.29kB / 2.29kB                                                                           0.0s
 => => sha256:dde0cca083bc75a0af14262b1469b5141284b4399a62fef923ec0c0e3b21f5bc 7.16kB / 7.16kB                                                                           0.0s
 => => sha256:2cc3ae149d28a36d28d4eefbae70aaa14a0c9eab588c3790f7979f310b893c44 29.15MB / 29.15MB                                                                         0.8s
 => => sha256:a97f9034bc9b7e813d93db97482046e20f581e1a80ddeda9b331c3ec6ed1cd8b 41.83MB / 41.83MB                                                                         1.2s
 => => sha256:24436676f2decbc5ed11c2e5786faa3dd103bc0fc738a2033b2f1aaab57226ad 398B / 398B                                                                               0.9s
 => => sha256:9571e65a55a3fd4ccd461b4fbaf5e8e38242317add94cb088268b70d6d7d08b2 627B / 627B                                                                               0.9s
 => => sha256:0b432cb2d95eea3d638db7e7cfb51eb7d7828f87c31d7a8c40ac5bb0278ca118 959B / 959B                                                                               0.9s
 => => extracting sha256:2cc3ae149d28a36d28d4eefbae70aaa14a0c9eab588c3790f7979f310b893c44                                                                                4.5s
 => => sha256:928cc9acedf0354de565f85d9df9d519e44a29a585d6c19a37a8aeb02e25212c 1.21kB / 1.21kB                                                                           1.0s
 => => sha256:ca6fb48c6db48342a3905bf65037e97543080a052a5f169b4b40b8c83b850f41 1.40kB / 1.40kB                                                                           1.0s
 => => extracting sha256:a97f9034bc9b7e813d93db97482046e20f581e1a80ddeda9b331c3ec6ed1cd8b                                                                                2.9s
 => => extracting sha256:9571e65a55a3fd4ccd461b4fbaf5e8e38242317add94cb088268b70d6d7d08b2                                                                                0.0s
 => => extracting sha256:0b432cb2d95eea3d638db7e7cfb51eb7d7828f87c31d7a8c40ac5bb0278ca118                                                                                0.0s
 => => extracting sha256:24436676f2decbc5ed11c2e5786faa3dd103bc0fc738a2033b2f1aaab57226ad                                                                                0.0s
 => => extracting sha256:928cc9acedf0354de565f85d9df9d519e44a29a585d6c19a37a8aeb02e25212c                                                                                0.0s
 => => extracting sha256:ca6fb48c6db48342a3905bf65037e97543080a052a5f169b4b40b8c83b850f41                                                                                0.0s
 => [internal] load build context                                                                                                                                        0.1s
 => => transferring context: 939.78kB                                                                                                                                    0.0s
 => [installer 2/6] WORKDIR /app                                                                                                                                         0.0s
 => [installer 3/6] COPY package*.json ./                                                                                                                                0.1s
 => [installer 4/6] RUN npm install                                                                                                                                     49.3s
 => [installer 5/6] COPY . .                                                                                                                                             0.1s
 => [installer 6/6] RUN npm run build                                                                                                                                   27.7s
 => [deployer 2/2] COPY --from=installer /app/build /usr/share/nginx/html                                                                                                0.1s
 => exporting to image                                                                                                                                                   0.0s
 => => exporting layers                                                                                                                                                  0.0s
 => => writing image sha256:09ae8f9d05aca8faa94bcc5eb42ea1a8e8eeb10b94431ef349462e97e028c04c                                                                             0.0s
 => => naming to docker.io/library/multi-stage 
```

- Let's see the image size
```console
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
multi-stage   latest    09ae8f9d05ac   41 seconds ago   189MB
```

- Run a container from the image
```console
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ docker run -it -d -p 3000:3000 --name for-fun multi-stage
e726d7446aad41a8a196231f4937ff72daec2e13d07a131b6df416b38545a420
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS         PORTS                            NAMES
e726d7446aad   multi-stage   "/docker-entrypoint.…"   6 seconds ago   Up 5 seconds   80/tcp, 0.0.0.0:3000->3000/tcp   for-fun
```

- If you need to view the log of the container because it may have some issues
```console
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ docker logs for-fun 
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2024/06/19 15:46:08 [notice] 1#1: using the "epoll" event method
2024/06/19 15:46:08 [notice] 1#1: nginx/1.27.0
2024/06/19 15:46:08 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14) 
2024/06/19 15:46:08 [notice] 1#1: OS: Linux 4.4.0-210-generic
2024/06/19 15:46:08 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2024/06/19 15:46:08 [notice] 1#1: start worker processes
2024/06/19 15:46:08 [notice] 1#1: start worker process 29
2024/06/19 15:46:08 [notice] 1#1: start worker process 30
2024/06/19 15:46:08 [notice] 1#1: start worker process 31
2024/06/19 15:46:08 [notice] 1#1: start worker process 32
2024/06/19 15:46:08 [notice] 1#1: start worker process 33
2024/06/19 15:46:08 [notice] 1#1: start worker process 34
2024/06/19 15:46:08 [notice] 1#1: start worker process 35
2024/06/19 15:46:08 [notice] 1#1: start worker process 36
```

- Going inside the container with sh
```console
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ docker exec -it for-fun sh
# id
uid=0(root) gid=0(root) groups=0(root)
```

- We are still inside the container and can check where we are. the issue we can find is we didn't set WORKDIR when we were build the image from stage two (deployer).
```console
# pwd
/
# ls -alh  
total 4.0K
drwxr-xr-x    1 root root   39 Jun 19 15:46 .
drwxr-xr-x    1 root root   39 Jun 19 15:46 ..
-rwxr-xr-x    1 root root    0 Jun 19 15:46 .dockerenv
lrwxrwxrwx    1 root root    7 Jun 12 00:00 bin -> usr/bin
drwxr-xr-x    2 root root    6 Jan 28 21:20 boot
drwxr-xr-x    5 root root  360 Jun 19 15:46 dev
drwxr-xr-x    1 root root   41 Jun 13 18:29 docker-entrypoint.d
-rwxrwxrwx    1 root root 1.6K Jun 13 18:28 docker-entrypoint.sh
drwxr-xr-x    1 root root   19 Jun 19 15:46 etc
drwxr-xr-x    2 root root    6 Jan 28 21:20 home
lrwxrwxrwx    1 root root    7 Jun 12 00:00 lib -> usr/lib
lrwxrwxrwx    1 root root    9 Jun 12 00:00 lib64 -> usr/lib64
drwxr-xr-x    2 root root    6 Jun 12 00:00 media
drwxr-xr-x    2 root root    6 Jun 12 00:00 mnt
drwxr-xr-x    2 root root    6 Jun 12 00:00 opt
dr-xr-xr-x 1216 root root    0 Jun 19 15:46 proc
drwx------    2 root root   37 Jun 12 00:00 root
drwxr-xr-x    1 root root   23 Jun 19 15:46 run
lrwxrwxrwx    1 root root    8 Jun 12 00:00 sbin -> usr/sbin
drwxr-xr-x    2 root root    6 Jun 12 00:00 srv
dr-xr-xr-x   13 root root    0 Mar  9 06:43 sys
drwxrwxrwt    2 root root    6 Jun 12 00:00 tmp
drwxr-xr-x    1 root root   19 Jun 12 00:00 usr
drwxr-xr-x    1 root root   19 Jun 12 00:00 var
# 
```

- Check what is in the directory of nginx
```console
# cd /usr/share/nginx/html  
# ls -ltr
total 44
-rw-r--r-- 1 root root  497 May 28 13:22 50x.html
-rw-r--r-- 1 root root 3870 Jun 19 15:42 favicon.ico
-rw-r--r-- 1 root root   67 Jun 19 15:42 robots.txt
-rw-r--r-- 1 root root  492 Jun 19 15:42 manifest.json
-rw-r--r-- 1 root root 9664 Jun 19 15:42 logo512.png
-rw-r--r-- 1 root root 5347 Jun 19 15:42 logo192.png
drwxr-xr-x 4 root root   27 Jun 19 15:43 static
-rw-r--r-- 1 root root  644 Jun 19 15:43 index.html
-rw-r--r-- 1 root root  517 Jun 19 15:43 asset-manifest.json
```
Note: Exit from container simply with Ctrl^D :)

- Inspect the container with docker inspect and we can see lots of details
```console
[node1] (local) root@192.168.0.8 /opt/todoapp-docker
$ docker inspect for-fun 
[
    {
        "Id": "e726d7446aad41a8a196231f4937ff72daec2e13d07a131b6df416b38545a420",
        "Created": "2024-06-19T15:46:07.30754639Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 9867,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2024-06-19T15:46:08.187858664Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:09ae8f9d05aca8faa94bcc5eb42ea1a8e8eeb10b94431ef349462e97e028c04c",
        "ResolvConfPath": "/var/lib/docker/containers/e726d7446aad41a8a196231f4937ff72daec2e13d07a131b6df416b38545a420/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/e726d7446aad41a8a196231f4937ff72daec2e13d07a131b6df416b38545a420/hostname",
        "HostsPath": "/var/lib/docker/containers/e726d7446aad41a8a196231f4937ff72daec2e13d07a131b6df416b38545a420/hosts",
        "LogPath": "/var/lib/docker/containers/e726d7446aad41a8a196231f4937ff72daec2e13d07a131b6df416b38545a420/e726d7446aad41a8a196231f4937ff72daec2e13d07a131b6df416b38545a4
20-json.log",
        "Name": "/for-fun",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "docker-default",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {
                "3000/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "3000"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "ConsoleSize": [
                36,
                174
            ],
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": [],
            "BlkioDeviceWriteBps": [],
            "BlkioDeviceReadIOps": [],
            "BlkioDeviceWriteIOps": [],
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware",
                "/sys/devices/virtual/powercap"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/7461cb96e13b9547b0bb8582830ed7535a7ed1fce673ff20ec91971cad8c0ae3-init/diff:/var/lib/docker/overlay2/qr6p3aj4j7ogp6ma5uptr2prv/diff:/var/lib/docker/overlay2/89a3ce54bd7e435ce2347dfe4bb52ddb17811fbc9fc332e8f09be09694a22e41/diff:/var/lib/docker/overlay2/4067d9b01a9abc35719ac4c1431a21d550ea2e7a84da83670d333961897b9f1d/diff:/var/lib/docker/overlay2/4a7ef1ef0ec499781b92cddf4aee52f82c845806013a6765bef88d3fae4e0b74/diff:/var/lib/docker/overlay2/727fdd566ca9cc66145b5f7a8cda01af6f24fc907ed54c06aa6e7fada36abd2c/diff:/var/lib/docker/overlay2/e1b862aaff039698244b46d4a6a41e469a27371d9dbb2501ad35d0e7074a9562/diff:/var/lib/docker/overlay2/c406ed264e0e46f8d290f603f350f1fb465a416f7500a98b4395b0a84690916e/diff:/var/lib/docker/overlay2/377199bc762e38cc7ef7949f3cd4fc80348870563f545384510aa55a69cd2db1/diff",
                "MergedDir": "/var/lib/docker/overlay2/7461cb96e13b9547b0bb8582830ed7535a7ed1fce673ff20ec91971cad8c0ae3/merged",
                "UpperDir": "/var/lib/docker/overlay2/7461cb96e13b9547b0bb8582830ed7535a7ed1fce673ff20ec91971cad8c0ae3/diff",
                "WorkDir": "/var/lib/docker/overlay2/7461cb96e13b9547b0bb8582830ed7535a7ed1fce673ff20ec91971cad8c0ae3/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "e726d7446aad",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3000/tcp": {},
                "80/tcp": {}
            },
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "NGINX_VERSION=1.27.0",
                "NJS_VERSION=0.8.4",
                "NJS_RELEASE=2~bookworm",
                "PKG_RELEASE=2~bookworm"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "multi-stage",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": [
                "/docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {
                "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
            },
            "StopSignal": "SIGQUIT"
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "68d106bf1259ed6eaf6787b43a7f80c765171b227c7ecc7e7731f5569977b6a8",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "3000/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "3000"
                    }
                ],
                "80/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/68d106bf1259",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "35da5057f13fade059cae7e2d3adfeeb6dcce89324ffc841b1c67b1761ac6105",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "be9af4aaabd83b6caeb1317b4a3f8b8441e89e2f22ea0b9f49e6d50ebf775777",
                    "EndpointID": "35da5057f13fade059cae7e2d3adfeeb6dcce89324ffc841b1c67b1761ac6105",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
```

- Some best practice for writing Dockerfile

1. Use Multi-stage Builds
2. Order Dockerfile Commands Appropriately
3. Use Small Docker Base Images
4. Minimize the Number of Layers
5. Use Unprivileged Containers
6. Prefer COPY Over ADD
7. Cache Python Packages to the Docker Host
8. Run Only One Process Per Container
9. Prefer Array Over String Syntax
10. Understand the Difference Between ENTRYPOINT and CMD
11. Include a HEALTHCHECK Instruction


