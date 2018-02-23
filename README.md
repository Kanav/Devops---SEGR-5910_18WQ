# Devops - Guestbook

## File Structure

```
guestbook/
 ├──public/                  * front end
 |   index.html              * index page html
 |   script.js               * All javascript functions
 |   style.css               * CSS styling for the app
 │
 |──main.go                      * required for executables.Program execution begins by initializing the main package and then invoking the function main
 │──guestbook-controller.json                 * create the guestbook replication controller
 |──guestbook-service.json                    * create a service to group the guestbook pods, to make the guestbook front end externally visible
 |──redis-master-controller.json              * replication controller to launch long-running pods
 ├──redis-slave-service.json                  * service to proxy connections to the read slaves
 ├──redis-master-service.json                 * service determines which pods will receive the traffic sent to the service
 └──redis-slave-controller.json               * responsible for managing the multiple instances of a replicated pod  

```

# Application will have the following components:

	Web front end
	Redis master for Storage (is used for write operations)
	Redis slave for offloading read operations


  I'm using containers because all of the application's code, libraries, and dependencies are packed together in the container as an immutable artifact.
  And using `Kubernetes` as an container orchestration platform, it allows large numbers of containers to work together in harmony, reducing operational burden. Other aspects it helps in are:
  *	Running containers across many different machines
  *	Scaling up or down by adding or removing containers when demand changes
  *	Keeping storage consistent with multiple instances of an application
  *	Distributing load between the containers
  *	Launching new containers on different machines if something fails


# Getting Started - Steps performed to build the application running at:
## http://35.226.72.24:3000
What you need to run this app:
* `google cloud SDK` and `gcloud` (`https://cloud.google.com/sdk/`)
* `Kubernetes`
* `Redis`



```bash
gcloud init
gcloud container clusters create guestbook
```

## Use Kubernetes
```bash
kubectl create -f redis-master-controller.json
kubectl get rc
NAME           DESIRED   CURRENT   READY     AGE
redis-master   1         1         0         28s

kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
redis-master-b64wd   1/1       Running   0          34s


kubectl create -f  redis-master-service.json
kubectl get services
NAME           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.15.240.1    <none>        443/TCP    3m
redis-master   ClusterIP   10.15.246.72   <none>        6379/TCP   25s

kubectl create -f redis-slave-controller.json
kubectl get rc
NAME           DESIRED   CURRENT   READY     AGE
redis-master   1         1         1         2m
redis-slave    2         2         0         25s



kubectl create -f redis-slave-service.json
kubectl get services
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.15.240.1     <none>        443/TCP    4m
redis-master   ClusterIP   10.15.246.72    <none>        6379/TCP   1m
redis-slave    ClusterIP   10.15.250.142   <none>        6379/TCP   26s

kubectl create -f guestbook-controller.json
kubectl get rc
NAME           DESIRED   CURRENT   READY     AGE
guestbook      3         3         3         25s
redis-master   1         1         1         3m
redis-slave    2         2         2         1m

kubectl get pods
NAME                 READY     STATUS    RESTARTS   AGE
guestbook-brj4f      1/1       Running   0          30s
guestbook-jcwgc      1/1       Running   0          30s
guestbook-jvkt5      1/1       Running   0          30s
redis-master-b64wd   1/1       Running   0          3m
redis-slave-9ltxv    1/1       Running   0          1m
redis-slave-v62wj    1/1       Running   0          1m

kubectl create -f guestbook-service.json
kubectl get services
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)          AGE
guestbook      LoadBalancer   10.15.253.234   35.226.72.24   3000:32277/TCP   3m
kubernetes     ClusterIP      10.15.240.1     <none>         443/TCP          7m
redis-master   ClusterIP      10.15.246.72    <none>         6379/TCP         4m
redis-slave    ClusterIP      10.15.250.142   <none>         6379/TCP         3m



```



## Running the app
After you have performed all the steps you can now run the app. The port will be displayed to you as `http://35.226.72.24:3000`.


## Other commands
```bash

# ssh-gcloud-cluster
kubectl get pgcloud compute ssh --zone us-central1-b guestbook
sudo docker ps

# redis-server
redis-server --slaveof redis-master 6379

```


## Documentation and Quickstart for Kubernetes
* https://cloud.google.com/kubernetes-engine/docs/quickstart/
