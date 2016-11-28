# Workshop: Deploying and Scaling Node.js with Kubernetes

Workshop at [Node Interactive North America 2016](http://events.linuxfoundation.org/events/node-interactive)

## Presenters
[Ross Kukulinski](https://twitter.com/rosskukulinski) and [Nathan White](https://twitter.com/_nw_)


## Workshop Description

As companies look to build out their next-generation architectures, Node.js and containerization are emerging as two major components for powering rapid technical innovation. In this technical workshop, we will show you how to get started with Node.js, Docker and Kubernetes and cover the pitfalls that often occur when starting and how to avoid them. Most of this workshop will be a live demonstration as we dockerize a Node.js application, deploy to Kubernetes, and scale to handle a large amount of traffic.


## Workshop Slides


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

```kubectl port-forward <pod id> 3000:3000```

```kubectl expose deployment frontend --port=80 --target-port=3000 --name=frontend```

```kubectl edit svc/frontend``` # Change to type: NodePort

```minikube service frontend```

```kubectl delete deployment/frontend service/frontend```


###  Deploy Frontend using declarative syntax

```less deploy/frontend.yml```

```kubectl create -f deploy/frontend.yml```

```kubectl get pods```


### Deploy API service using declarative syntax

[API Service](https://github.com/cloudytimemachine/api) depends on Redis and RethinkDB, so lets create them first.

```kubectl less deploy/redis.yml```

```kubectl create -f deploy/redis.yml```

```kubectl get pods```

```kubectl port-forward <redis pod> 6379:6379```

```redis-cli ```

Kubectl can also create from URL (Similar to  `curl | bash`)

```
kubectl create -f https://raw.githubusercontent.com/rosskukulinski/kubernetes-rethinkdb-cluster/master/rethinkdb-quickstart.yml
```

```kubectl port-forward <rethinkdb admin> 8080:8080```

```open localhost:8080```

```less deploy/api.yml```

```kubectl create -f deploy/api.yml```

```kubectl get pods```

```kubectl get svc```

```kubectl logs <api pod> -f```

```kubectl logs <api pod> -f | bunyan```


### Deploy Gateway service to expose frontend & api
