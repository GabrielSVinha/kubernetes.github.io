---
title: Setting OpenStack as a Kubernetes Cloud Provider
---

## Overview

This guide shows you how to setup your cluster using OpenStack as cloud provider.
Some of the advantages of using OpenStack as cloud provider are:

- Allows you to create Load Balancer services in a On Premise cluster,
- Allows you to create volumes to persistency in your cluster,
- You can configure your provider manually.

## Requirements

You need a up and running cluster and access (via SSH) to the cluster nodes, if you have any trouble in this step, check these [steps](http://henriquetruta.com/2017/09/07/creating-kubernetes-cluster/).

Also, you need an OpenStack cloud which is reachable by Kubernetes. This can be done using Keystone for authentication, then being able to communicate with Neutron component. If you need assistance with OpenStack resources (we will go through some of them here), check out this [link](https://docs.openstack.org/heat/latest/template_guide/openstack.html).

## Connect Cloud to Master Node

### Cloud Configuration File

We want to let our Kubernetes Cluster to use our OpenStack instance as Cloud Provider, for doing so we need to add a configuration file to our master node.

We call this file `cloud.conf` and it will have the following structure:

```
[Global]
username=user  
password=pass  
auth-url=https://<openstack_endpoint>:5000/v3  
tenant-id=c869168a828847f39f7f06edd7305637  
domain-id=2a73b8f597c04551a0fdc8e95544be8a

[LoadBalancer]
subnet-id=6937f8fa-858d-4bc9-a3a5-18d2c957166a  
floating-network-id=9cdf8226-fd6c-499a-994e-d12e51a498af 
```

Where:

- `username` and `password` are authentication credential used by Keystone to access the cloud resources,
- `auth-url` is the keystone endpoint. Find it in horizon in Access and Security > API Access > Credentials,
- `tenant-id` is the id of the project (tenant) you want to create your resources on,
- `domain-id` is the id of the domain your user belongs to,
- `subnet-id` is the id of the subnet you want to create your loadbalancer on. Get it in Network > Networks and click on the respective network to get its subnets,
- `floating-network-id` is the id of the network you want to create your floating IP on. Emit this option if you don't want it.

### Other Values

The `cloud.conf` file supports other values, one example is

```
[BlockStorage]
bs-version=v2
```

Which tells Cinder to provide volumes to the Kubernetes cluster.

#### Transfering configuration information to master node

The above OpenStack information needs to be located in the cluster master node, specifically in `/etc/kubernetes`, copy it by running:

```
scp cloud.conf user@<master-ip-address>:/etc/kubernetes/
```

### Adding configuration to kube-controller-manager

Now that you have the credentials file inside the node, we need to make **kube-controller-manager** manifest to point to the file.

For that, connect to the machine (I strongly recommend using SSH) and edit the file located at `/etc/kubernetes/manifests/kube-controller-manager.yaml`. Navigate to the `spec > containers > command` section and add two new flags:

```
--cloud-provider=openstack
--cloud-config=/etc/kubernetes/cloud.conf
```