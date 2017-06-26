---
title: "Example: Deploying WordPress and MySQL with Persistent Volumes"
assignees:
- ahmetb
---

<<<<<<< HEAD
{% capture overview %}
This tutorial shows you how to deploy a WordPress site and a MySQL database using Minikube. Both applications use PersistentVolumes and PersistentVolumeClaims to store data. 

A [PersistentVolume](/docs/concepts/storage/persistent-volumes/) (PV) is a piece of storage in the cluster that has been provisioned by an administrator, and a [PeristentVolumeClaim](/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) (PVC) is a set amount of storage in a PV. PersistentVolumes and PeristentVolumeClaims are independent from Pod lifecycles and preserve data through restarting, rescheduling, and even deleting Pods. 

**Warning:**  This deployment is not suitable for production use cases, as it uses single instance WordPress and MySQL Pods. Consider using [WordPress Helm Chart](https://github.com/kubernetes/charts/tree/master/stable/wordpress) to deploy WordPress in production.
{: .warning}

{% endcapture %}

{% capture objectives %}
* Create a PersistentVolume
* Create a Secret
* Deploy MySQL
* Deploy WordPress
* Clean up

{% endcapture %}

{% capture prerequisites %}

{% include task-tutorial-prereqs.md %} 

Download the following configuration files:

1. [local-volumes.yaml](/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/local-volumes.yaml)

1. [mysql-deployment.yaml](/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/mysql-deployment.yaml)

1. [wordpress-deployment.yaml](/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/wordpress-deployment.yaml)

{% endcapture %}

{% capture lessoncontent %} 

## Create a PersistentVolume

MySQL and Wordpress each use a PersistentVolume to store data. While Kubernetes supports many different [types of PersistentVolumes](/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes), this tutorial covers [hostPath](/docs/concepts/storage/volumes/#hostpath).

**Note:** If you have a Kubernetes cluster running on Google Container Engine, please follow [this guide](https://cloud.google.com/container-engine/docs/tutorials/persistent-disk).
{: .note}

### Setting up a hostPath Volume

A `hostPath` mounts a file or directory from the host node’s filesystem into your Pod. 

**Warning:** Only use `hostPath` for developing and testing. With hostPath, your data lives on the node the Pod is scheduled onto and does not move between nodes. If a Pod dies and gets scheduled to another node in the cluster, the data is lost. 
{: .warning}

1. Launch a terminal window in the directory you downloaded the manifest files.

2. Create two PersistentVolumes from the `local-volumes.yaml` file:

       kubectl create -f local-volumes.yaml

{% include code.html language="yaml" file="mysql-wordpress-persistent-volume/local-volumes.yaml" ghlink="/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/local-volumes.yaml" %}

{:start="3"} 
3. Run the following command to verify that two 20GiB PersistentVolumes are available:

       kubectl get pv

   The response should be like this:

       NAME         CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
       local-pv-1   20Gi       RWO           Retain          Available                                      1m
       local-pv-2   20Gi       RWO           Retain          Available                                      1m

## Create a Secret for MySQL Password

A [Secret](/docs/concepts/configuration/secret/) is an object that stores a piece of sensitive data like a password or key. The manifest files are already configured to use a Secret, but you have to create your own Secret.

1. Create the Secret object from the following command:

       kubectl create secret generic mysql-pass --from-literal=password=YOUR_PASSWORD
       
   **Note:** Replace `YOUR_PASSWORD` with the password you want to apply.     
   {: .note}
   
2. Verify that the Secret exists by running the following command:

       kubectl get secrets

   The response should be like this:

       NAME                  TYPE                                  DATA      AGE
       mysql-pass                 Opaque                                1         42s

   **Note:** To protect the Secret from exposure, neither `get` nor `describe` show its contents. 
   {: .note}

## Deploy MySQL

The following manifest describes a single-instance MySQL Deployment. The MySQL container mounts the PersistentVolume at /var/lib/mysql. The `MYSQL_ROOT_PASSWORD` environment variable sets the database password from the Secret. 

{% include code.html language="yaml" file="mysql-wordpress-persistent-volume/mysql-deployment.yaml" ghlink="/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/mysql-deployment.yaml" %}

1. Deploy MySQL from the `mysql-deployment.yaml` file:

       kubectl create -f mysql-deployment.yaml

2. Verify that the Pod is running by running the following command:

       kubectl get pods

   **Note:** It can take up to a few minutes for the Pod's Status to be `RUNNING`.
   {: .note}

   The response should be like this:

       NAME                               READY     STATUS    RESTARTS   AGE
       wordpress-mysql-1894417608-x5dzt   1/1       Running   0          40s

## Deploy WordPress

The following manifest describes a single-instance WordPress Deployment and Service. It uses many of the same features like a PVC for persistent storage and a Secret for the password. But it also uses a different setting: `type: NodePort`. This setting exposes WordPress to traffic from outside of the cluster.

{% include code.html language="yaml" file="mysql-wordpress-persistent-volume/wordpress-deployment.yaml" ghlink="/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/wordpress-deployment.yaml" %}

1. Create a WordPress Service and Deployment from the `wordpress-deployment.yaml` file:
=======
This example describes how to run a persistent installation of
[WordPress](https://wordpress.org/) and
[MySQL](https://www.mysql.com/) on Kubernetes. We'll use the
[mysql](https://registry.hub.docker.com/_/mysql/) and
[wordpress](https://registry.hub.docker.com/_/wordpress/) official
[Docker](https://www.docker.com/) images for this installation. (The
WordPress image includes an Apache server).

Demonstrated Kubernetes Concepts:

* [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) to
  define persistent disks (disk lifecycle not tied to the Pods).
* [Services](https://kubernetes.io/docs/concepts/services-networking/service/) to enable Pods to
  locate one another.
* [External Load Balancers](https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer)
  to expose Services externally.
* [Deployments](http://kubernetes.io/docs/user-guide/deployments/) to ensure Pods
  stay up and running.
* [Secrets](http://kubernetes.io/docs/user-guide/secrets/) to store sensitive
  passwords.

## Quickstart

Put your desired MySQL password in a file called `password.txt` with
no trailing newline. The first `tr` command will remove the newline if
your editor added one.

**Note:** if your cluster enforces **_selinux_** and you will be using [Host Path](#host-path) for storage, then please follow this [extra step](#selinux).

```shell
tr --delete '\n' <password.txt >.strippedpassword.txt && mv .strippedpassword.txt password.txt
kubectl create -f https://raw.githubusercontent.com/kubernetes/examples/master/mysql-wordpress-pd/local-volumes.yaml
kubectl create secret generic mysql-pass --from-file=password.txt
kubectl create -f https://raw.githubusercontent.com/kubernetes/examples/master/mysql-wordpress-pd/mysql-deployment.yaml
kubectl create -f https://raw.githubusercontent.com/kubernetes/examples/master/mysql-wordpress-pd/wordpress-deployment.yaml
```

## Table of Contents

<!-- BEGIN MUNGE: GENERATED_TOC -->

- [Persistent Installation of MySQL and WordPress on Kubernetes](#persistent-installation-of-mysql-and-wordpress-on-kubernetes)
  - [Quickstart](#quickstart)
  - [Table of Contents](#table-of-contents)
  - [Cluster Requirements](#cluster-requirements)
  - [Decide where you will store your data](#decide-where-you-will-store-your-data)
    - [Host Path](#host-path)
        - [SELinux](#selinux)
    - [GCE Persistent Disk](#gce-persistent-disk)
  - [Create the MySQL Password Secret](#create-the-mysql-password-secret)
  - [Deploy MySQL](#deploy-mysql)
  - [Deploy WordPress](#deploy-wordpress)
  - [Visit your new WordPress blog](#visit-your-new-wordpress-blog)
  - [Take down and restart your blog](#take-down-and-restart-your-blog)
  - [Next Steps](#next-steps)

<!-- END MUNGE: GENERATED_TOC -->

## Cluster Requirements

Kubernetes runs in a variety of environments and is inherently
modular. Not all clusters are the same. These are the requirements for
this example.

* Kubernetes version 1.2 is required due to using newer features, such
  at PV Claims and Deployments. Run `kubectl version` to see your
  cluster version.
* [Cluster DNS](https://github.com/kubernetes/dns) will be used for service discovery.
* An [external load balancer](https://kubernetes.io/docs/concepts/services-networking/service/#type-loadbalancer)
  will be used to access WordPress.
* [Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)
  are used. You must create Persistent Volumes in your cluster to be
  claimed. This example demonstrates how to create two types of
  volumes, but any volume is sufficient.

Consult a
[Getting Started Guide](http://kubernetes.io/docs/getting-started-guides/)
to set up a cluster and the
[kubectl](http://kubernetes.io/docs/user-guide/prereqs/) command-line client.

## Decide where you will store your data

MySQL and WordPress will each use a
[Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
to store their data. We will use a Persistent Volume Claim to claim an
available persistent volume. This example covers HostPath and
GCEPersistentDisk volumes. Choose one of the two, or see
[Types of Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)
for more options.

### Host Path

Host paths are volumes mapped to directories on the host. **These
should be used for testing or single-node clusters only**. The data
will not be moved between nodes if the pod is recreated on a new
node. If the pod is deleted and recreated on a new node, data will be
lost.

##### SELinux

On systems supporting selinux it is preferred to leave it enabled/enforcing.
However, docker containers mount the host path with the "_svirt_sandbox_file_t_"
label type, which is incompatible with the default label type for /tmp ("_tmp_t_"),
resulting in a permissions error when the mysql container attempts to `chown`
_/var/lib/mysql_.
Therefore, on selinx systems using host path, you should pre-create the host path
directory (/tmp/data/) and change it's selinux label type to "_svirt_sandbox_file_t_",
as follows:

```shell
## on every node:
mkdir -p /tmp/data
chmod a+rwt /tmp/data  # match /tmp permissions
chcon -Rt svirt_sandbox_file_t /tmp/data
```

Continuing with host path, create the persistent volume objects in Kubernetes using
[local-volumes.yaml](https://git.k8s.io/examples/mysql-wordpress-pd/local-volumes.yaml):

```shell
export KUBE_REPO=https://raw.githubusercontent.com/kubernetes/examples/master
kubectl create -f $KUBE_REPO/mysql-wordpress-pd/local-volumes.yaml
```


### GCE Persistent Disk

This storage option is applicable if you are running on
[Google Compute Engine](http://kubernetes.io/docs/getting-started-guides/gce/).

Create two persistent disks. You will need to create the disks in the
same [GCE zone](https://cloud.google.com/compute/docs/zones) as the
Kubernetes cluster. The default setup script will create the cluster
in the `us-central1-b` zone, as seen in the
[config-default.sh](https://git.k8s.io/kubernetes/cluster/gce/config-default.sh) file. Replace
`<zone>` below with the appropriate zone. The names `wordpress-1` and
`wordpress-2` must match the `pdName` fields we have specified in
[gce-volumes.yaml](https://git.k8s.io/examples/mysql-wordpress-pd/gce-volumes.yaml).

```shell
gcloud compute disks create --size=20GB --zone=<zone> wordpress-1
gcloud compute disks create --size=20GB --zone=<zone> wordpress-2
```

Create the persistent volume objects in Kubernetes for those disks:

```shell
export KUBE_REPO=https://raw.githubusercontent.com/kubernetes/examples/master
kubectl create -f $KUBE_REPO/mysql-wordpress-pd/gce-volumes.yaml
```

## Create the MySQL Password Secret

Use a [Secret](http://kubernetes.io/docs/user-guide/secrets/) object
to store the MySQL password. First create a file (in the same directory
as the wordpress sample files) called
`password.txt` and save your password in it. Make sure to not have a
trailing newline at the end of the password. The first `tr` command
will remove the newline if your editor added one. Then, create the
Secret object.

```shell
tr --delete '\n' <password.txt >.strippedpassword.txt && mv .strippedpassword.txt password.txt
kubectl create secret generic mysql-pass --from-file=password.txt
```

This secret is referenced by the MySQL and WordPress pod configuration
so that those pods will have access to it. The MySQL pod will set the
database password, and the WordPress pod will use the password to
access the database.

## Deploy MySQL

Now that the persistent disks and secrets are defined, the Kubernetes
pods can be launched. Start MySQL using
[mysql-deployment.yaml](https://git.k8s.io/examples/mysql-wordpress-pd/mysql-deployment.yaml).

```shell
kubectl create -f $KUBE_REPO/mysql-wordpress-pd/mysql-deployment.yaml
```

Take a look at [mysql-deployment.yaml](https://git.k8s.io/examples/mysql-wordpress-pd/mysql-deployment.yaml), and
note that we've defined a volume mount for `/var/lib/mysql`, and then
created a Persistent Volume Claim that looks for a 20G volume. This
claim is satisfied by any volume that meets the requirements, in our
case one of the volumes we created above.

Also look at the `env` section and see that we specified the password
by referencing the secret `mysql-pass` that we created above. Secrets
can have multiple key:value pairs. Ours has only one key
`password.txt` which was the name of the file we used to create the
secret. The [MySQL image](https://hub.docker.com/_/mysql/) sets the
database password using the `MYSQL_ROOT_PASSWORD` environment
variable.

It may take a short period before the new pod reaches the `Running`
state.  List all pods to see the status of this new pod.

```shell
kubectl get pods
```

```
NAME                          READY     STATUS    RESTARTS   AGE
wordpress-mysql-cqcf4-9q8lo   1/1       Running   0          1m
```

Kubernetes logs the stderr and stdout for each pod. Take a look at the
logs for a pod by using `kubectl log`. Copy the pod name from the
`get pods` command, and then:

```shell
kubectl logs <pod-name>
```

```
...
2016-02-19 16:58:05 1 [Note] InnoDB: 128 rollback segment(s) are active.
2016-02-19 16:58:05 1 [Note] InnoDB: Waiting for purge to start
2016-02-19 16:58:05 1 [Note] InnoDB: 5.6.29 started; log sequence number 1626007
2016-02-19 16:58:05 1 [Note] Server hostname (bind-address): '*'; port: 3306
2016-02-19 16:58:05 1 [Note] IPv6 is available.
2016-02-19 16:58:05 1 [Note]   - '::' resolves to '::';
2016-02-19 16:58:05 1 [Note] Server socket created on IP: '::'.
2016-02-19 16:58:05 1 [Warning] 'proxies_priv' entry '@ root@wordpress-mysql-cqcf4-9q8lo' ignored in --skip-name-resolve mode.
2016-02-19 16:58:05 1 [Note] Event Scheduler: Loaded 0 events
2016-02-19 16:58:05 1 [Note] mysqld: ready for connections.
Version: '5.6.29'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server (GPL)
```

Also in [mysql-deployment.yaml](https://git.k8s.io/examples/mysql-wordpress-pd/mysql-deployment.yaml) we created a
service to allow other pods to reach this mysql instance. The name is
`wordpress-mysql` which resolves to the pod IP.

Up to this point one Deployment, one Pod, one PVC, one Service, one Endpoint,
two PVs, and one Secret have been created, shown below:

```shell
kubectl get deployment,pod,svc,endpoints,pvc -l app=wordpress -o wide && \
  kubectl get secret mysql-pass && \
  kubectl get pv
```

```shell
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deploy/wordpress-mysql   1         1         1            1           3m
NAME                                  READY     STATUS    RESTARTS   AGE       IP           NODE
po/wordpress-mysql-3040864217-40soc   1/1       Running   0          3m        172.17.0.2   127.0.0.1
NAME                  CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE       SELECTOR
svc/wordpress-mysql   None         <none>        3306/TCP   3m        app=wordpress,tier=mysql
NAME                 ENDPOINTS         AGE
ep/wordpress-mysql   172.17.0.2:3306   3m
NAME                 STATUS    VOLUME       CAPACITY   ACCESSMODES   AGE
pvc/mysql-pv-claim   Bound     local-pv-2   20Gi       RWO           3m
NAME         TYPE      DATA      AGE
mysql-pass   Opaque    1         3m
NAME         CAPACITY   ACCESSMODES   STATUS      CLAIM                    REASON    AGE
local-pv-1   20Gi       RWO           Available                                      3m
local-pv-2   20Gi       RWO           Bound       default/mysql-pv-claim             3m
```

## Deploy WordPress

Next deploy WordPress using
[wordpress-deployment.yaml](https://git.k8s.io/examples/mysql-wordpress-pd/wordpress-deployment.yaml):
>>>>>>> Update links to proper repos

       kubectl create -f wordpress-deployment.yaml

2. Verify that the Service is running by running the following command:

       kubectl get services wordpress

   The response should be like this:

       NAME        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
       wordpress   10.0.0.89    <pending>     80:32406/TCP   4m

   **Note:** Minikube can only expose Services through `NodePort`. <br/><br/>The `EXTERNAL-IP` is always `<pending>`.
   {: .note}

3. Run the following command to get the IP Address for the WordPress Service:

       minikube service wordpress --url

   The response should be like this:

       http://1.2.3.4:32406

4. Copy the IP address, and load the page in your browser to view your site.

   You should see the WordPress set up page similar to the following screenshot.

   ![wordpress-init](https://raw.githubusercontent.com/kubernetes/examples/master/mysql-wordpress-pd/WordPress.png)

   **Warning:** Do not leave your WordPress installation on this page. If another user finds it, they can set up a website on your instance and use it to serve malicious content. <br/><br/>Either install WordPress by creating a username and password or delete your instance.
   {: .warning}

{% endcapture %}

{% capture cleanup %}

1. Run the following command to delete your Secret:

       kubectl delete secret mysql-pass

2. Run the following commands to delete all Deployments and Services:

       kubectl delete deployment -l app=wordpress
       kubectl delete service -l app=wordpress

3. Run the following commands to delete the PersistentVolumeClaims and the PersistentVolumes:

       kubectl delete pvc -l app=wordpress
       kubectl delete pv local-pv-1 local-pv-2
       
   **Note:** Any other Type of PersistentVolume would allow you to recreate the Deployments and Services at this point without losing data, but `hostPath` loses the data as soon as the Pod stops running.
   {: .note}         

{% endcapture %}

{% capture whatsnext %}

* Learn more about [Introspection and Debugging](/docs/tasks/debug-application-cluster/debug-application-introspection/)
* Learn more about [Jobs](/docs/concepts/workloads/controllers/jobs-run-to-completion/)
* Learn more about [Port Forwarding](/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)
* Learn how to [Get a Shell to a Container](/docs/tasks/debug-application-cluster/get-shell-running-container/)

{% endcapture %}

{% include templates/tutorial.md %}
