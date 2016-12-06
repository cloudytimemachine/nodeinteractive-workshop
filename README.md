# Workshop: Deploying and Scaling Node.js with Kubernetes

Workshop at [Node Interactive North America 2016](http://events.linuxfoundation.org/events/node-interactive)

## Presenters
[Ross Kukulinski](https://twitter.com/rosskukulinski) and [Nathan White](https://twitter.com/_nw_)


## Workshop Description

As companies look to build out their next-generation architectures, Node.js and containerization are emerging as two major components for powering rapid technical innovation. In this technical workshop, we will show you how to get started with Node.js, Docker and Kubernetes and cover the pitfalls that often occur when starting and how to avoid them. Most of this workshop will be a live demonstration as we dockerize a Node.js application, deploy to Kubernetes, and scale to handle a large amount of traffic.


## Workshop Slides

[Workshop Slides - TBD](http://www.slideshare.net/kuxman/workshop-deploying-and-scaling-nodejs-with-kubernetes)  
[Workshop Screencast - TBD]()

## Background Reading / Videos

https://vimeo.com/172626818
http://kubernetes.io/docs/hellonode/
http://www.slideshare.net/kuxman/nodejs-and-containers-go-together-like-peanut-butter-and-jelly

## Outline

* Welcome & Introductions
* Installing minikube & kubectl
* Kubernetes Overview
* Cloudy Time Machine (CTM) Overview
* Deploy CTM to minikube / Kubernetes

## Deploying Cloudy Time Machine to Kubernetes

### Working with minikube

```minikube start```

```minikube stop```

```minikube service <servicename>```

```minikube dashboard```

```minikube ip```

```minikube docker-env```

```minikube logs```


### Working with kubectl

```kubectl config current-context```

```kubectl config use-context minikube```

```kubectl get nodes```

```kubectl describe node <nodeid>```

```kubectl get pods```

```kubectl get namespaces```

```kubectl -n kube-system get pods```

```kubectl -n kube-system get services```

```kubectl -n kube-system get svc```

```kubectl -n kube-system describe svc kubernetes-dashboard```

```kubectl run -i -t busybox --image=busybox --restart=Never```


### Deploy CTM frontend manually

The CTM [frontend](https://github.com/cloudytimemachine/frontend) application can run standalone, so let's start with that.

```kubectl run frontend --image=quay.io/cloudytimemachine/frontend:25f9df2e9c991745a02ef5fdd249a542c53d6d8a --image-pull-policy=IfNotPresent```

```kubectl get pods```

```kubectl get events --watch```

```kubectl describe pod <pod id>```

```kubectl port-forward <pod id> 3000:80```

```curl localhost:3000```

```kubectl exec -ti <frontend pod> /bin/sh```

```vi index.html```

```curl localhost:3000```

```kubectl expose deployment frontend --port=80 --target-port=3000 --name=frontend```

```kubectl describe svc frontend```

```kubectl scale deployment/frontend --replicas=2```

```kubectl describe svc frontend```

```kubectl edit pod <frontend pod>``` # Edit label to remove from the pool

```kubectl get pods```

```kubectl edit svc/frontend``` # Change to type: NodePort

```minikube service frontend```

```kubectl delete deployment/frontend svc/frontend```

```kubectl delete pod <frontend pod>```


###  Deploy Frontend using declarative syntax

```atom deploy/frontend.yml```

```kubectl create -f deploy/frontend.yml```

```kubectl get pods```

```kubectl get svc```


###  Remind audience about architecture

![Insert architecture image here]()


### Deploy API service using declarative syntax

[API Service](https://github.com/cloudytimemachine/api) depends on Redis and RethinkDB, so lets create them first.

```atom deploy/redis.yml```

```kubectl create -f deploy/redis.yml```

```kubectl get pods```

```kubectl port-forward <redis pod> 6379:6379```

```redis-cli ```

Kubectl can also create from URL (Similar to  `curl | bash`)

```kubectl create -f https://raw.githubusercontent.com/rosskukulinski/kubernetes-rethinkdb-cluster/master/rethinkdb-quickstart.yml```

```kubectl get pods --watch```

```minikube service rethinkdb-admin```

```atom deploy/api.yml```

```kubectl create -f deploy/api.yml```

```kubectl get pods```

```kubectl get svc```

```kubectl logs <api pod> --tail=10 -f```

```kubectl logs <api pod> --tail=10 -f | bunyan```


### Deploy Gateway service to expose frontend & api

```atom deploy/gateway.yml```

```kubectl create -f deploy/gateway.yml```

```kubectl get configmap```

```kubectl get pods```

```kubectl describe svc gateway```

```minikube service gateway```

Navigate to /api/snapshots --> See empty array

Request snapshot for xkcd.com, gets queued in Redis

### Deploy worker-screenshot service

```atom deploy/worker-screenshot.yml```

```kubectl create -f deploy/worker-screenshot.yml```

```kubetctl get pods --watch```

```kubectl logs <worker pod> -f | bunyan```

Request additional snapshots, see the queue size increase

```kubectl scale deployment/worker-screenshot --replicas=3```

```kubectl get pods --watch```

### Tear everything down

```kubectl delete namespace default --cascade=true```

### Bring it back up

```
make
kubectl create -f deploy/all-in-one.yml
```

### Back to slides to wrap up
