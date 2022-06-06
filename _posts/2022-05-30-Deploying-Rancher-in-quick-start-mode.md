---
layout: post
title: Deploying Rancher for Demo and Testing
date: '2022-05-30T12:47:00.000+05:30'
author: Middlewarebytes
sitemap:
  lastmod: 2022-05-31
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
tags:
- Kubernetes
- Docker
- Rancher
- New technology
- Cloud native
readtime: true
share-title: How to deploy a rancher instance for testing.
---

By now I think most of us would have heard already heard about  Kubernetes and in some manner have worked with it. I am not going to  write anything about Kubernetes here but a software which I see a lot of organizations have started  using and in some ways it compliments a lot to the k8s stack we have been using in our teams.


**_A quick intro on Rancher_**

-----------------------------------

Nowdays, it is not unusual for a company to run a handful of k8s clusters with each cluster having special configuration and access-control settings, sometimes these clusters hosted on different cloud stacks or datacenters, Rancher can effectively manage all such k8s clusters acting as a single pan of glass for sorts of administration and management activities. 

With its management UI, users can make broad changes to a cluster or a group of clusters from a central location.
In my team as well, while doing a POC about  self-managed K8s cluster and its manageability, Rancher sat perfectly well on all our checkboxes 😆
What I just mentioned is just the beginning, Rancher has much more to offer. Checkout the official documentation for more detailed information.

Here , I will only install Rancher in a quick start mode which is  helpful  if you  are trying out Rancher for the first time but note that this method of installing Rancher is just for test or demo purposes only. If you are planning to use Rancher for development or production uses ,go and see the appropriate topologies for installation. I am also not  going to create or import a k8s cluster here, I will leave that for later.


### Pre-requisites:

Install docker on the host machine (where you are going to install rancher).


**Note**: If your plan is to create a k8s cluster using k3s or RKE, you need to install docker on all the machines which would be used for cluster nodes as well.

Rancher should work with any modern Linux distribution,  I have used Ubuntu  because it is my favourite 😀 ,we can follow the official docker installation document itself for installing docker. This is the Ubuntu [official](https://docs.docker.com/engine/install/ubuntu/) documentation , similar is available for CentOs,debian or other distros.


### Installing Rancher


1. Connect to the VM using a shell / client of your choice such as PuTTy or xterm. 

2. Run the following command from your shell:

    `sudo docker run -d --restart=unless-stopped -p 8080:80 -p 8443:443 --privileged rancher/rancher`

3. Wait for a few seconds and you should see the Rancher server container up and running by running the below command:

    `docker ps`
 
4. Log in to the Rancher UI using **_<VM-IP>:<PORT>_** where _PORT_ is the mapped port given above in the Docker run command. If everything is up and running, one should see the below screen in the browser on hitting the URL.

     ![Howdy! Welcome to Rancher](/img/postimages/rancherui.jpg?raw=true "Rancher")


5. The default password can be changed from the one-time password generated for the login by following the on-screen instructions.

     ![Set your password](/img/postimages/rancherlogincredentials.jpg?raw=true "Password")

