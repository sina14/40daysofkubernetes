## Day 2/40
# How To Dockerize a Project
[Video Link](https://www.youtube.com/watch?v=nfRsPiRGx74)
@piyushsachdeva 

We started with cloning a git repository from docker. It's a simple node application.

```console
root@192.168.0.8 ~/day02
$ git clone https://github.com/docker/getting-started-app.git
Cloning into 'getting-started-app'...
remote: Enumerating objects: 79, done.
remote: Counting objects: 100% (28/28), done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 79 (delta 17), reused 17 (delta 13), pack-reused 51
Receiving objects: 100% (79/79), 1.76 MiB | 12.86 MiB/s, done.
Resolving deltas: 100% (18/18), done.
```

Secondly, Login with a credential on docker hub
```console
root@192.168.0.8 ~
$ docker login
Log in with your Docker ID or email address to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com/ to create one.
You can log in with your password or a Personal Access Token (PAT). Using a limited-scope PAT grants better security and is required for organizations using SSO. Learn more at https://docs.docker.com/go/access-tokens/

Username: sinatavakkol
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

Third, We wrote a Dockerfile
```
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

Forth, Build the image
```console
root@192.168.0.8 ~/day02/getting-started-app
$ sudo docker build -t day02-todo .
[+] Building 23.2s (10/10) FINISHED                                                                                                                            docker:default
 => [internal] load .dockerignore                                                                                                                                        0.0s
 => => transferring context: 64B                                                                                                                                         0.0s
 => [internal] load build definition from Dockerfile                                                                                                                     0.0s
 => => transferring dockerfile: 155B                                                                                                                                     0.0s
 => [internal] load metadata for docker.io/library/node:18-alpine                                                                                                        0.4s
 => [auth] library/node:pull token for registry-1.docker.io                                                                                                              0.0s
 => [1/4] FROM docker.io/library/node:18-alpine@sha256:6937be95129321422103452e2883021cc4a96b63c32d7947187fcb25df84fc3f                                                  4.3s
 => => resolve docker.io/library/node:18-alpine@sha256:6937be95129321422103452e2883021cc4a96b63c32d7947187fcb25df84fc3f                                                  0.0s
 => => sha256:6937be95129321422103452e2883021cc4a96b63c32d7947187fcb25df84fc3f 1.43kB / 1.43kB                                                                           0.0s
 => => sha256:05412f5b9ed819c373a2535804e473a155fc91bfb7adf469ec2312e056a9e87f 1.16kB / 1.16kB                                                                           0.0s
 => => sha256:e7d39d4d8569a6203be5b7a118d4d92526b267087023a49ee0868f7c50190191 7.23kB / 7.23kB                                                                           0.0s
 => => sha256:d25f557d7f31bf7acfac935859b5153da41d13c41f2b468d16f729a5b883634f 3.62MB / 3.62MB                                                                           0.1s
 => => sha256:f6124930634921d33d69a1a8b5848cb40d0b269e79b4c37c236cb5e4d61a2710 39.83MB / 39.83MB                                                                         1.0s
 => => sha256:22a81a0f8d1c30ce5a5da3579a84ab4c22fd2f14cb33863c1a752da6f056dc18 1.38MB / 1.38MB                                                                           0.1s
 => => extracting sha256:d25f557d7f31bf7acfac935859b5153da41d13c41f2b468d16f729a5b883634f                                                                                0.2s
 => => sha256:bd06542006fda4279cb2edd761a84311c1fdbb90554e9feaaf078a3674845742 447B / 447B                                                                               0.2s
 => => extracting sha256:f6124930634921d33d69a1a8b5848cb40d0b269e79b4c37c236cb5e4d61a2710                                                                                2.9s
 => => extracting sha256:22a81a0f8d1c30ce5a5da3579a84ab4c22fd2f14cb33863c1a752da6f056dc18                                                                                0.1s
 => => extracting sha256:bd06542006fda4279cb2edd761a84311c1fdbb90554e9feaaf078a3674845742                                                                                0.0s
 => [internal] load build context                                                                                                                                        0.1s
 => => transferring context: 6.47MB                                                                                                                                      0.1s
 => [2/4] WORKDIR /app                                                                                                                                                   0.0s
 => [3/4] COPY . .                                                                                                                                                       0.1s
 => [4/4] RUN yarn install --production                                                                                                                                 15.8s
 => exporting to image                                                                                                                                                   2.4s 
 => => exporting layers                                                                                                                                                  2.4s 
 => => writing image sha256:41ea09f4ff8628eb00c044f4f5402238c5eb2815b281e998a7a74ca9ca3d1abf                                                                             0.0s 
 => => naming to docker.io/library/day02-todo
```

Fifth, Tag the image
```console
docker tag day02-todo:latest sinatavakkol/40daysofkube:02.0
```

Sixth, Push the image to docker hub
```console
docker push sinatavakkol/40daysofkube:02.0
```
![image](https://github.com/sina14/40daysofkubernetes/assets/8893590/1ae1abb2-ee51-4577-b396-a7775bb92b6e)

7th, Run and create the container
```console
docker run -d -p 3000:3000 --name fina sinatavakkol/40daysofkube:02.0
```
