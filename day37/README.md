## Day 37/40
# Application Failure Troubleshooting From CKA
[Video Link](https://www.youtube.com/watch?v=Mil0HUtPg6I)
@piyushsachdeva 
[Git Repository](https://github.com/piyushsachdeva/CKA-2024/)
[My Git Repo](https://github.com/sina14/40daysofkubernetes)


In this section, we're looking at application failures.

We have a sample app for instance

```sh
git clone https://github.com/piyushsachdeva/example-voting-app.git

```

As it mentioned in the source [repository](https://github.com/piyushsachdeva/example-voting-app):

- A front-end web app in Python which lets you vote between two options
- A Redis which collects new votes
- A .NET worker which consumes votes and stores them in…
- A Postgres database backed by a Docker volume
- A Node.js web app which shows the results of the voting in real time

```sh
root@sinaops:/opt/example-voting-app# docker compose up -d
...
root@sinaops:/opt/example-voting-app# docker compose ps
NAME                          IMAGE                       COMMAND                  SERVICE   CREATED          STATUS                    PORTS
example-voting-app-db-1       postgres:15-alpine          "docker-entrypoint.s…"   db        32 seconds ago   Up 31 seconds (healthy)   5432/tcp
example-voting-app-redis-1    redis:alpine                "docker-entrypoint.s…"   redis     32 seconds ago   Up 31 seconds (healthy)   6379/tcp
example-voting-app-result-1   example-voting-app-result   "nodemon --inspect=0…"   result    32 seconds ago   Up 25 seconds             127.0.0.1:9229->9229/tcp, 0.0.0.0:5001->80/tcp, :::5001->80/tcp
example-voting-app-vote-1     example-voting-app-vote     "python app.py"          vote      32 seconds ago   Up 25 seconds (healthy)   0.0.0.0:5000->80/tcp, :::5000->80/tcp
example-voting-app-worker-1   example-voting-app-worker   "dotnet Worker.dll"      worker    32 seconds ago   Up 26 seconds

```

Run in `Kubernetes`:

```sh
root@sinaops:/opt/example-voting-app# kubectl apply -f  k8s-specifications/
deployment.apps/db created
service/db created
networkpolicy.networking.k8s.io/access-redis created
deployment.apps/redis created
service/redis created
deployment.apps/result created
service/result created
deployment.apps/vote created
service/vote created
deployment.apps/worker created

```

```sh
root@sinaops:/opt/example-voting-app# kubectl get pods,deploy,svc -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
pod/db-597b4ff8d7-h4flp       1/1     Running   0          21m   10.85.0.10   cloudy.net   <none>           <none>
pod/redis-796dc594bb-dgglw    1/1     Running   0          21m   10.85.0.11   cloudy.net   <none>           <none>
pod/result-d8c4c69b8-ffc8n    1/1     Running   0          21m   10.85.0.16   cloudy.net   <none>           <none>
pod/vote-69cb46f6fb-ln4np     1/1     Running   0          21m   10.85.0.12   cloudy.net   <none>           <none>
pod/worker-5dd767667f-4csr5   1/1     Running   0          21m   10.85.0.15   cloudy.net   <none>           <none>
pod/worker-5dd767667f-l2589   1/1     Running   0          21m   10.85.0.13   cloudy.net   <none>           <none>
pod/worker-5dd767667f-m6xk2   1/1     Running   0          21m   10.85.0.14   cloudy.net   <none>           <none>

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                  SELECTOR
deployment.apps/db       1/1     1            1           21m   postgres     postgres:15-alpine                      app=db
deployment.apps/redis    1/1     1            1           21m   redis        redis:alpine                            app=redis
deployment.apps/result   1/1     1            1           21m   result       dockersamples/examplevotingapp_result   app=result
deployment.apps/vote     1/1     1            1           21m   vote         dockersamples/examplevotingapp_vote     app=vote
deployment.apps/worker   3/3     3            3           21m   worker       dockersamples/examplevotingapp_worker   app=worker

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE   SELECTOR
service/db           ClusterIP   10.110.219.117   <none>        5432/TCP         21m   app=db
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          2d    <none>
service/redis        ClusterIP   10.109.149.22    <none>        6379/TCP         21m   app=redis
service/result       NodePort    10.100.83.247    <none>        5001:31001/TCP   21m   app=results
service/vote         NodePort    10.98.179.36     <none>        5000:31000/TCP   21m   app=vote

```








