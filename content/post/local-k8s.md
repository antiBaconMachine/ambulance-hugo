---
subtitle: "Or having your cake and eating it"
date: 2020-06-11T17:28:24Z
tags: []
---
# Local Kubernetes Clusters For Development

In this post I'm going to talk about running kubernetes locally on a dev machine. Specifically I'm going to cover

* Why you might want to do this
* What the options are
* Making the dev experience not suck

## Why?

Kubernetes is a sledgehammer, your dev environment is a nut. These two things do not work well together. If you can avoid running kubernetes locally on a day to day basis you absolutely should. Feel free to stop reading now.

There are good reasons for doing this however, perhaps you want to debug a specific integration issue between your software and the kubernetes platform. In my case I stared down this route because for the last few months I've been building a system designed from the ground up to run i kubernetes, it involves running many batch jobs in a cluster on a schedule using CronJobs. Whilst the individual jobs can be built and tested somewhat in isolation the only way to know if the whole system is working is to run the thing in kubernetes. This means we need a lightweight install for dev and test environments.

## What are the options

I investigated a few different local solutions, here are my very brief notes

* Minikube
  - Been around the longest
  - Requires a VM
* Docker Desktop
  - Bundled with software you may already have
  - The knarly internals are quite hidden away, which may or may not be a good thing
  - Runs in containers, no VM required
* (k3s)[https://k3s.io/]/(k3d)[https://k3d.io/]
  - k3s is a lightweight kubernetes install provided by Rancher
  - k3d is a cli tool for managing k3s clusters in docker containers

For my purposes it's quite important that my solution is easily scriptable so it can be shared with my whole team. It's certainly possible to script all of these, but for me k3d seems to be the lightest touch, I just have to get people to install one new tool. Minikube might require a whole vm infrastructure, docker for desktop might require some manual intervention in a ui to enable the cluster.

## Making the dev experience not suck

To get every dev a cluster in as painless a way as possible I wrote a simple bootstrap script to ge the cluster up. It's designed to be idempotant so it can be run safely multiple times.

```
#! /usr/bin/env bash

# Suffix the cluster with the k3s version we're using
CLUSTER_NAME=antibacon-1-17-3

# Check if we have previously created a cluster with this name
k3d ls | awk '{print $2}' | grep $CLUSTER_NAME
CLUSTER_DOES_NOT_EXIST=$?

set -eu

if [ $CLUSTER_DOES_NOT_EXIST -eq 1 ]; then
    # cluster does not exist, create it
    k3d create \
      --name $CLUSTER_NAME \
      --enable-registry \
      --api-port 6555 \
      --image rancher/k3s:v1.17.3-k3s1 \
      --wait 0 \
      --workers 2 \
      --publish 8888:80
else
    # cluster already exists, start it
    k3d start -n $CLUSTER_NAME
fi

```

Other things which may be appropriate in this script would be setting up any pull secrets you need, for now I'll keep this simple.

```
❯./bootstrap.sh
INFO[0000] Created cluster network with ID fec90ee806c8492eed8da445ebfedd7b732aafcda7be054664374cd2673acb02
INFO[0000] Created docker volume  k3d-antibacon-1-17-3-images
INFO[0000] Creating cluster [antibacon-1-17-3]
INFO[0000] Registry already present: ensuring that it's running and connecting it to the 'k3d-antibacon-1-17-3' network...
INFO[0000] Creating server using docker.io/rancher/k3s:v1.17.3-k3s1...
INFO[0009] Booting 2 workers for cluster antibacon-1-17-3
INFO[0009] Created worker with ID 512b6c0da3f602158902106b1884eb9514a65d8a5f2f802c3fdedb2d873c3947
INFO[0010] Created worker with ID 1a7fcdc54929b031624892db985d1ad15f573d6f3c1e010141735dd777bf33cc
INFO[0010] SUCCESS: created cluster [antibacon-1-17-3]
INFO[0010] A local registry has been started as registry.local:5000
INFO[0010] You can now use the cluster with:

export KUBECONFIG="$(k3d get-kubeconfig --name='antibacon-1-17-3')"
kubectl cluster-info
```

Assuming you already have docker running and k3d installed, this script will stand up a 3 cluster kubernetes instance in docker. Pretty cool. The last line of output is important, users must point their local kubectl at the new cluster.

You might want to invest in a automated way of managing context switching. The way I like to achieve this is with (direnv)[https://direnv.net/], a great tool which allows you to set environment variables on a per directory basis. For our purposes we create a file `.envrc` with the line from the above output:

```
#! /usr/bin/env bash
export KUBECONFIG="$(k3d get-kubeconfig --name='antibacon-1-17-3')"
```

and now whenever we are in our directory direnv will manage context switching for us.

### Using the cluster

To actually deploy something to the cluster we can use kubectl.

```
❯kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
deployment.apps/hello-node created
❯kubectl expose deployment hello-node --type=LoadBalancer --port=8080
service/hello-node exposed
```

However to actually get to this service we have to set up a port forward

```
kubectl port-forward svc/hello-node 8080:8080
```

Which is not particularly user friendly. What we really want is a public ingress. This is actually pretty straightforward

```
cat <<EOF | kubectl apply -f-
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-node
spec:
  rules:
   - http:
      paths:
      - path: /hello
        backend:
          serviceName: hello-node
          servicePort: 8080
EOF
```

and now we can `curl localhost:7777/hello` to reach the service. 7777 is an arbitrary port we published in the bootstrap script which maps to port 80 on the cluster.

For host based ingress you can either add each host you want to map to your hosts file or you can run a local dns service such as (dnsmasq)[http://www.thekelleys.org.uk/dnsmasq/doc.html] to map an entire subdomain or tld to your localhost.

### Running your own images

The kubernetes cluster itself needs to be able to fetch any images it needs. This is a problem if you want to run some development code. To help with this when we created our cluster earlier we used the k3d `--enable-registry` flag which sets up a local docker registry accessible both to our local machine and the cluster.

To use it first make sure registry.local maps to 127.0.0.1. You can do this either via your hosts file or with dnsmasq. Now we can push any image as `registry.local:5000/myimage` and both our local docker daemon and the cluster will understand. For example let's just try aliasing the echoserver we've been using as a local image.

```
# first we create a deployment
❯kubectl create deployment hello-dev --image=registry.local:5000/echoserver
deployment.apps/hello-dev created

# The image we provided won't be found so this pod will never start
❯kubectl get po | grep hello-dev
hello-dev-6bbb5969cf-k67qs    0/1     ImagePullBackOff   0          82s

# Pull the echoserver image. Note that our local docker daemon doesn't already have this even though we've been using the image in our cluster
❯docker pull  k8s.gcr.io/echoserver:1.4

# tag the echoserver image for our local registry
❯docker tag k8s.gcr.io/echoserver:1.4 registry.local:5000/echoserver:latest

# push the newly tagged image to the registry
❯docker push registry.local:5000/echoserver:latest

# The next time the deployment tries to pull the image it will be available so we shold eventually see the container create ok
❯kubectl get po | grep hello-dev
hello-dev-6bbb5969cf-lkwn7   1/1     Running   0          9s
```

### Getting something else to do all this for you

There are a few tools which attempt to automate a lot of this dev cycle pain. The only one I'm reasonably familiar with is (skaffold)[https://skaffold.dev/]. It certainly helps with the local image build and push loop and it has a nice image rewriting feature so you don't have to rename all your images `registry.local:5000/whatever` in dev.

## Conclusions

It is not at all simple to provide a pleasant kubernetes experience locally but it's getting easier all the time. I stress again, I would not reccomend you undertake a project like this unless you have a good reason to. If you just want to learn kubernetes then I'd reccomend using one of the many online tools whcih provide you with a ready made test cluster. If you really need to do this then as long as you are prepared to put up with the tool soup (`docker`, `kubectl`, `k3d`, `direnv`, `dnsmasq`, `skaffold`), not to mention some light shell scripting, then you can eveuntually assemble a reasonable dev experience.