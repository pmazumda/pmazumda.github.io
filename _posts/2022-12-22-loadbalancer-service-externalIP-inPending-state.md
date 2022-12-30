---
layout: post
title:  Troubleshooting Azure AKS Loadbalancer Service Creation
date: '2022-12-22T18:47:00.000+05:30'
author: Middlewarebytes
sitemap:
  lastmod: 2022-12-30
  priority: 0.7
  changefreq: 'monthly'
  exclude: 'no'
tags:
- Kubernetes
- New technology
- Cloud native
- Azure
- Networking
- AKS

readtime: true
---


Many a times while we see the external IP for a service as type LoadBalancer in PENDING state. If we look at  the events of the service we could see messages similar to this :

![Events](/img/postimages/troubleshootingloadbalancerservicecreation-1.png?raw=true "Events")

This is because the cluster identity or the service principal for the AKS cluster does not have access to the resources which it is trying to look for in the resource group.In this case the service principal for the subjected cluster does not have access to read the vnet resources (subnet) under the __resource__group__ virtualnetworks-><VNET>/subnets/<SUBNETNAME>. 


To validate, follow the below steps.

Go to the Azure Portal -> Kubernetes Services -> Click on its Resource Group.
Click on its Managed Identity, check Azure Role Assignments. The managed identity should have the Contributor role over the Virtual Network resource type.


Refer: [Configure Azure CNI networking in Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/aks/configure-azure-cni#advanced-networking-prerequisites){:target="_blank"}
