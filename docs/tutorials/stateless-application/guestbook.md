---
title: "Example: Deploying PHP Guestbook application with Redis"
assignees:
- ahmetb
---

{% capture overview %}
This tutorial shows you how to build and deploy a simple, multi-tier web application using Kubernetes and [Docker](https://www.docker.com/). This example consists of the following components:

* A single-instance [Redis](https://redis.io/) master to store guestbook entries 
* Multiple replicated Redis instances to serve reads 
* Multiple web frontend instances

{% endcapture %}

{% capture objectives %}
* Start up a Redis master.
* Start up Redis slaves.
* Start up the guestbook frontend.
* Expose and view the Frontend Service.
* Clean up.
{% endcapture %}

{% capture prerequisites %}

{% include task-tutorial-prereqs.md %}
Download the following configuration files:

1. [redis-master-deployment.yaml](/docs/tutorials/stateless-application/guestbook/redis-master-deployment.yaml)
1. [redis-master-service.yaml](/docs/tutorials/stateless-application/guestbook/redis-master-service.yaml)
1. [redis-slave-deployment.yaml](/docs/tutorials/stateless-application/guestbook/redis-slave-deployment.yaml)
1. [redis-slave-service.yaml](/docs/tutorials/stateless-application/guestbook/redis-slave-service.yaml)
1. [frontend-deployment.yaml](/docs/tutorials/stateless-application/guestbook/frontend-deployment.yaml)
1. [frontend-service.yaml](/docs/tutorials/stateless-application/guestbook/frontend-service.yaml)

{% endcapture %}

{% capture lessoncontent %}

## Start up the Redis Master

The guestbook application uses Redis to store its data. It writes its data to a Redis master instance and reads data from multiple Redis slave instances.

### Creating the Redis Master Deployment

<<<<<<< HEAD
The manifest file, included below, specifies a Deployment controller that runs a single replica Redis master Pod.
=======
If you see a url response, you are ready to go. If not, read the [Getting Started guides](http://kubernetes.io/docs/getting-started-guides/) for how to get started, and follow the [prerequisites](http://kubernetes.io/docs/user-guide/prereqs/) to install and configure `kubectl`. As noted above, if you have a Google Container Engine cluster set up, read [this example](https://cloud.google.com/container-engine/docs/tutorials/guestbook) instead.

All the files referenced in this example can be downloaded [from GitHub](https://git.k8s.io/examples/guestbook).

### Quick Start

This section shows the simplest way to get the example work. If you want to know the details, you should skip this and read [the rest of the example](#step-one-start-up-the-redis-master).

Start the guestbook with one command:

```console
$ kubectl create -f guestbook/all-in-one/guestbook-all-in-one.yaml
service "redis-master" created
deployment "redis-master" created
service "redis-slave" created
deployment "redis-slave" created
service "frontend" created
deployment "frontend" created
```
>>>>>>> Update links to proper repos

1. Launch a terminal window in the directory you downloaded the manifest files.
2. Apply the Redis Master Deployment from the `redis-master-deployment.yaml` file:

       kubectl apply -f redis-master-deployment.yaml
        
   {% include code.html language="yaml" file="guestbook/redis-master-deployment.yaml" ghlink="/docs/tutorials/stateless-application/guestbook/redis-master-deployment.yaml" %}

3. Query the list of Pods to verify that the Redis Master Pod is running:

<<<<<<< HEAD
       kubectl get pods
=======
```console
$ kubectl get services
NAME           CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
frontend       10.0.0.117   <none>        80/TCP     20s
redis-master   10.0.0.170   <none>        6379/TCP   20s
redis-slave    10.0.0.201   <none>        6379/TCP   20s
```

Now you can access the guestbook on each node with frontend Service's `<Cluster-IP>:<PORT>`, e.g. `10.0.0.117:80` in this guide. `<Cluster-IP>` is a cluster-internal IP. If you want to access the guestbook from outside of the cluster, add `type: NodePort` to the frontend Service `spec` field. Then you can access the guestbook with `<NodeIP>:NodePort` from outside of the cluster. On cloud providers which support external load balancers, adding `type: LoadBalancer` to the frontend Service `spec` field will provision a load balancer for your Service. There are several ways for you to access the guestbook. You may learn from [Accessing services running on the cluster](https://kubernetes.io/docs/concepts/cluster-administration/access-cluster/#accessing-services-running-on-the-cluster).

Clean up the guestbook:

```console
$ kubectl delete -f guestbook/all-in-one/guestbook-all-in-one.yaml
```

or

```console
$ kubectl delete -f guestbook/
```


### Step One: Start up the redis master

Before continuing to the gory details, we also recommend you to read Kubernetes [concepts and user guide](http://kubernetes.io/docs/user-guide/).
**Note**: The redis master in this example is *not* highly available.  Making it highly available would be an interesting, but intricate exercise — redis doesn't actually support multi-master Deployments at this point in time, so high availability would be a somewhat tricky thing to implement, and might involve periodic serialization to disk, and so on.

#### Define a Deployment

To start the redis master, use the file [redis-master-deployment.yaml](https://git.k8s.io/examples/guestbook/redis-master-deployment.yaml), which describes a single [pod](http://kubernetes.io/docs/user-guide/pods/) running a redis key-value server in a container.

Although we have a single instance of our redis master, we are using a [Deployment](http://kubernetes.io/docs/user-guide/deployments/) to enforce that exactly one pod keeps running. E.g., if the node were to go down, the Deployment will ensure that the redis master gets restarted on a healthy node. (In our simplified example, this could result in data loss.)

The file [redis-master-deployment.yaml](redis-master-deployment.yaml) defines the redis master Deployment:

<!-- BEGIN MUNGE: EXAMPLE redis-master-deployment.yaml -->

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: redis-master
  # these labels can be applied automatically 
  # from the labels in the pod template if not set
  # labels:
  #   app: redis
  #   role: master
  #   tier: backend
spec:
  # this replicas value is default
  # modify it according to your case
  replicas: 1
  # selector can be applied automatically 
  # from the labels in the pod template if not set
  # selector:
  #   matchLabels:
  #     app: guestbook
  #     role: master
  #     tier: backend
  template:
    metadata:
      labels:
        app: redis
        role: master
        tier: backend
    spec:
      containers:
      - name: master
        image: gcr.io/google_containers/redis:e2e
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```
>>>>>>> Update links to proper repos

   The response should be similar to this:

       NAME                            READY     STATUS    RESTARTS   AGE
       redis-master-1068406935-3lswp   1/1       Running   0          28s

4. Run the following command to view the logs from the Redis Master Pod:

       kubectl logs -f POD-NAME

<<<<<<< HEAD
**Note:** Replace POD-NAME with the name of your Pod.
{: .note}
=======
The file [redis-master-service.yaml](https://git.k8s.io/examples/guestbook/redis-master-deployment.yaml) defines the redis master Service:
>>>>>>> Update links to proper repos

### Creating the Redis Master Service

The guestbook applications needs to communicate to the Redis master to write its data. You need to apply a [Service](/docs/concepts/services-networking/service/) to proxy the traffic to the Redis master Pod. A Service defines a policy to access the Pods.

1. Apply the Redis Master Service from the following `redis-master-service.yaml` file: 

       kubectl apply -f redis-master-service.yaml

   {% include code.html language="yaml" file="guestbook/redis-master-service.yaml" ghlink="/docs/tutorials/stateless-application/guestbook/redis-master-service.yaml" %}

**Note:** This manifest file creates a Service named `redis-master` with a set of labels that match the labels previously defined, so the Service routes network traffic to the Redis master Pod.   
{: .note}

2. Query the list of Services to verify that the Redis Master Service is running:

       kubectl get service

   The response should be similar to this:

       NAME           CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
       kubernetes     10.0.0.1     <none>        443/TCP    1m
       redis-master   10.0.0.151   <none>        6379/TCP   8s

## Start up the Redis Slaves

Although the Redis master is a single pod, you can make it highly available to meet traffic demands by adding replica Redis slaves.

### Creating the Redis Slave Deployment

<<<<<<< HEAD
Deployments scale based off of the configurations set in the manifest file. In this case, the Deployment object specifies two replicas. 
=======
If your cluster does not have the DNS service enabled, then you can use environment variables by setting the
`GET_HOSTS_FROM` env value in both
[redis-slave-deployment.yaml](https://git.k8s.io/examples/guestbook/redis-slave-deployment.yaml) and [frontend-deployment.yaml](https://git.k8s.io/examples/guestbook/frontend-deployment.yaml)
from `dns` to `env` before you start up the app.
(However, this is unlikely to be necessary. You can check for the DNS service in the list of the cluster's services by
running `kubectl --namespace=kube-system get rc -l k8s-app=kube-dns`.)
Note that switching to env causes creation-order dependencies, since Services need to be created before their clients that require env vars.
>>>>>>> Update links to proper repos

If there are not any replicas running, this Deployment would start the two replicas on your container cluster. Conversely, if there are more than two replicas are running, it would scale down until two replicas are running. 

1. Apply the Redis Slave Deployment from the `redis-slave-deployment.yaml` file:

       kubectl apply -f redis-slave-deployment.yaml

   {% include code.html language="yaml" file="guestbook/redis-slave-deployment.yaml" ghlink="/docs/tutorials/stateless-application/guestbook/redis-slave-deployment.yaml" %}

2. Query the list of Pods to verify that the Redis Slave Pods are running:

       kubectl get pods

   The response should be similar to this:

       NAME                            READY     STATUS              RESTARTS   AGE
       redis-master-1068406935-3lswp   1/1       Running             0          1m
       redis-slave-2005841000-fpvqc    0/1       ContainerCreating   0          6s
       redis-slave-2005841000-phfv9    0/1       ContainerCreating   0          6s
        
### Creating the Redis Slave Service

The guestbook application needs to communicate to Redis slaves to read data. To make the Redis slaves discoverable, you need to set up a Service. A Service provides transparent load balancing to a set of Pods.

1. Apply the Redis Slave Service from the following `redis-slave-service.yaml` file:

       kubectl apply -f redis-slave-service.yaml

   {% include code.html language="yaml" file="guestbook/redis-slave-service.yaml" ghlink="/docs/tutorials/stateless-application/guestbook/redis-slave-service.yaml" %}

2. Query the list of Services to verify that the Redis Slave Service is running:

       kubectl get services

   The response should be similar to this:

       NAME           CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
       kubernetes     10.0.0.1     <none>        443/TCP    2m
       redis-master   10.0.0.151   <none>        6379/TCP   1m
       redis-slave    10.0.0.223   <none>        6379/TCP   6s

## Set up and Expose the Guestbook Frontend

The guestbook application has a web frontend serving the HTTP requests written in PHP. It is configured to connect to the `redis-master` Service for write requests and the `redis-slave` service for Read requests.

### Creating the Guestbook Frontend Deployment

1. Apply the frontend Deployment from the following `frontend-deployment.yaml` file:

       kubectl apply -f frontend-deployment.yaml

   {% include code.html language="yaml" file="guestbook/frontend-deployment.yaml" ghlink="/docs/tutorials/stateless-application/guestbook/frontend-deployment.yaml" %}

2. Query the list of Pods to verify that the three frontend replicas are running:

       kubectl get pods -l app=guestbook -l tier=frontend

   The response should be similar to this:

       NAME                        READY     STATUS    RESTARTS   AGE
       frontend-3823415956-dsvc5   1/1       Running   0          54s
       frontend-3823415956-k22zn   1/1       Running   0          54s
       frontend-3823415956-w9gbt   1/1       Running   0          54s

### Creating the Frontend Service

The `redis-slave` and `redis-master` Services you applied are only accessible within the container cluster because the default type for a Service is [ClusterIP](/docs/concepts/services-networking/service/#publishing-services---service-types). `ClusterIP` provides a single IP address for the set of Pods the Service is pointing to. This IP address is accessible only within the cluster.

If you want guests to be able to access your guestbook, you must configure the frontend Service to be externally visible, so a client can request the Service from outside the container cluster. Minikube can only expose Services through `NodePort`.  

**Note:** Some cloud providers, like Google Compute Engine or Google Container Engine, support external load balancers. If your cloud provider supports load balancers and you want to use it, simply delete or comment out `type: NodePort`, and uncomment `type: LoadBalancer`. 
{: .note}

<<<<<<< HEAD
1. Apply the frontend Service from the following `frontend-service.yaml` file:
=======
This time we put the Service and Deployment into one [file](http://kubernetes.io/docs/user-guide/managing-deployments/#organizing-resource-configurations). Grouping related objects together in a single file is often better than having separate files.
The specification for the slaves is in [all-in-one/redis-slave.yaml](https://git.k8s.io/examples/guestbook/all-in-one/redis-slave.yaml):
>>>>>>> Update links to proper repos

       kubectl apply -f frontend-service.yaml
        
   {% include code.html language="yaml" file="guestbook/frontend-service.yaml" ghlink="/docs/tutorials/stateless-application/guestbook/frontend-service.yaml" %}

<<<<<<< HEAD
2. Query the list of Services to verify that the frontend Service is running:

       kubectl get services 

   The response should be similar to this:

       NAME           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
       frontend       10.0.0.112   <nodes>       80:31323/TCP   6s
       kubernetes     10.0.0.1     <none>        443/TCP        4m
       redis-master   10.0.0.151   <none>        6379/TCP       2m
       redis-slave    10.0.0.223   <none>        6379/TCP       1m

### Viewing the Frontend Service via `NodePort`

If you deployed this application to Minikube or a local cluster, you need to find the IP address to view your Guestbook.

1. Run the following command to get the IP address for the frontend Service.

       minikube service frontend --url

   The response should be similar to this:
=======
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: redis
    role: slave
    tier: backend
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
  selector:
    app: redis
    role: slave
    tier: backend
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: redis-slave
  # these labels can be applied automatically
  # from the labels in the pod template if not set
  # labels:
  #   app: redis
  #   role: slave
  #   tier: backend
spec:
  # this replicas value is default
  # modify it according to your case
  replicas: 2
  # selector can be applied automatically
  # from the labels in the pod template if not set
  # selector:
  #   matchLabels:
  #     app: guestbook
  #     role: slave
  #     tier: backend
  template:
    metadata:
      labels:
        app: redis
        role: slave
        tier: backend
    spec:
      containers:
      - name: slave
        image: gcr.io/google_samples/gb-redisslave:v1
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access an environment variable to find the master
          # service's host, comment out the 'value: dns' line above, and
          # uncomment the line below.
          # value: env
        ports:
        - containerPort: 6379
```

[Download example](https://raw.githubusercontent.com/kubernetes/examples/master/guestbook/all-in-one/redis-slave.yaml)
<!-- END MUNGE: EXAMPLE all-in-one/redis-slave.yaml -->

This time the selector for the Service is `app=redis,role=slave,tier=backend`, because that identifies the pods running redis slaves. It is generally helpful to set labels on your Service itself as we've done here to make it easy to locate them with the `kubectl get services -l "app=redis,role=slave,tier=backend"` command. For more information on the usage of labels, see [using-labels-effectively](http://kubernetes.io/docs/user-guide/managing-deployments/#using-labels-effectively).

Now that you have created the specification, create the Service in your cluster by running:

```console
$ kubectl create -f guestbook/all-in-one/redis-slave.yaml
service "redis-slave" created
deployment "redis-slave" created

$ kubectl get services
NAME           CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
redis-master   10.0.76.248    <none>        6379/TCP   20m
redis-slave    10.0.112.188   <none>        6379/TCP   16s

$ kubectl get deployments
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
redis-master   1         1         1            1           22m
redis-slave    2         2         2            2           2m
```

Once the Deployment is up, you can list the pods in the cluster, to verify that the master and slaves are running.  You should see a list that includes something like the following:

```console
$ kubectl get pods
NAME                            READY     STATUS    RESTARTS   AGE
redis-master-2353460263-1ecey   1/1       Running   0          35m
redis-slave-1691881626-dlf5f    1/1       Running   0          15m
redis-slave-1691881626-sfn8t    1/1       Running   0          15m
```

You should see a single redis master pod and two redis slave pods.  As mentioned above, you can get more information about any pod with: `kubectl describe pods/<POD_NAME>`. And also can view the resources on [kube-ui](http://kubernetes.io/docs/user-guide/ui/).

### Step Three: Start up the guestbook frontend

A frontend pod is a simple PHP server that is configured to talk to either the slave or master services, depending on whether the client request is a read or a write. It exposes a simple AJAX interface, and serves an Angular-based UX.
Again we'll create a set of replicated frontend pods instantiated by a Deployment — this time, with three replicas.

As with the other pods, we now want to create a Service to group the frontend pods.
The Deployment and Service are described in the file [all-in-one/frontend.yaml](https://git.k8s.io/examples/guestbook/all-in-one/frontend.yaml):

<!-- BEGIN MUNGE: EXAMPLE all-in-one/frontend.yaml -->

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # if your cluster supports it, uncomment the following to automatically create
  # an external load-balanced IP for the frontend service.
  # type: LoadBalancer
  ports:
    # the port that this service should serve on
  - port: 80
  selector:
    app: guestbook
    tier: frontend
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
  # these labels can be applied automatically
  # from the labels in the pod template if not set
  # labels:
  #   app: guestbook
  #   tier: frontend
spec:
  # this replicas value is default
  # modify it according to your case
  replicas: 3
  # selector can be applied automatically
  # from the labels in the pod template if not set
  # selector:
  #   matchLabels:
  #     app: guestbook
  #     tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google-samples/gb-frontend:v4
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        env:
        - name: GET_HOSTS_FROM
          value: dns
          # If your cluster config does not include a dns service, then to
          # instead access environment variables to find service host
          # info, comment out the 'value: dns' line above, and uncomment the
          # line below.
          # value: env
        ports:
        - containerPort: 80
```

[Download example](https://raw.githubusercontent.com/kubernetes/examples/master/guestbook/all-in-one/frontend.yaml)
<!-- END MUNGE: EXAMPLE all-in-one/frontend.yaml -->

#### Using 'type: LoadBalancer' for the frontend service (cloud-provider-specific)

For supported cloud providers, such as Google Compute Engine or Google Container Engine, you can specify to use an external load balancer
in the service `spec`, to expose the service onto an external load balancer IP.
To do this, uncomment the `type: LoadBalancer` line in the [all-in-one/frontend.yaml](https://git.k8s.io/examples/guestbook/all-in-one/frontend.yaml) file before you start the service.

[See the appendix below](#appendix-accessing-the-guestbook-site-externally) on accessing the guestbook site externally for more details.

Create the service and Deployment like this:

```console
$ kubectl create -f guestbook/all-in-one/frontend.yaml
service "frontend" created
deployment "frontend" created
```

Then, list all your services again:

```console
$ kubectl get services
NAME           CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
frontend       10.0.63.63     <none>        80/TCP     1m
redis-master   10.0.76.248    <none>        6379/TCP   39m
redis-slave    10.0.112.188   <none>        6379/TCP   19m
```

Also list all your Deployments:

```console
$ kubectl get deployments 
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
frontend       3         3         3            3           2m
redis-master   1         1         1            1           39m
redis-slave    2         2         2            2           20m
```

Once it's up, i.e. when desired replicas match current replicas (again, it may take up to thirty seconds to create the pods), you can list the pods with specified labels in the cluster, to verify that the master, slaves and frontends are all running. You should see a list containing pods with label 'tier' like the following:

```console
$ kubectl get pods -L tier
NAME                            READY     STATUS    RESTARTS   AGE       TIER
frontend-1211764471-4e1j2       1/1       Running   0          4m        frontend
frontend-1211764471-gkbkv       1/1       Running   0          4m        frontend
frontend-1211764471-rk1cf       1/1       Running   0          4m        frontend
redis-master-2353460263-1ecey   1/1       Running   0          42m       backend
redis-slave-1691881626-dlf5f    1/1       Running   0          22m       backend
redis-slave-1691881626-sfn8t    1/1       Running   0          22m       backend
```

You should see a single redis master pod, two redis slaves, and three frontend pods.

The code for the PHP server that the frontends are running is in `examples/guestbook/php-redis/guestbook.php`.  It looks like this:

```php
<?

set_include_path('.:/usr/local/lib/php');

error_reporting(E_ALL);
ini_set('display_errors', 1);

require 'Predis/Autoloader.php';

Predis\Autoloader::register();

if (isset($_GET['cmd']) === true) {
  $host = 'redis-master';
  if (getenv('GET_HOSTS_FROM') == 'env') {
    $host = getenv('REDIS_MASTER_SERVICE_HOST');
  }
  header('Content-Type: application/json');
  if ($_GET['cmd'] == 'set') {
    $client = new Predis\Client([
      'scheme' => 'tcp',
      'host'   => $host,
      'port'   => 6379,
    ]);
>>>>>>> Update links to proper repos

       http://192.168.99.100:31323

2. Copy the IP address, and load the page in your browser to view your guestbook.

### Viewing the Frontend Service via `LoadBalancer`

If you deployed the `frontend-service.yaml` manifest with type: `LoadBalancer` you need to find the IP address to view your Guestbook.

1. Run the following command to get the IP address for the frontend Service.

       kubectl get service frontend

   The response should be similar to this:

       NAME       CLUSTER-IP      EXTERNAL-IP        PORT(S)        AGE
       frontend   10.51.242.136   109.197.92.229     80:32372/TCP   1m

2. Copy the External IP address, and load the page in your browser to view your guestbook.

## Scale the Web Frontend 

Scaling up or down is easy because your servers are defined as a Service that uses a Deployment controller.

1. Run the following command to scale up the number of frontend Pods:

       kubectl scale deployment frontend --replicas=5

2. Query the list of Pods to verify the number of frontend Pods running:

       kubectl get pods

   The response should look similar to this: 

       NAME                            READY     STATUS    RESTARTS   AGE
       frontend-3823415956-70qj5       1/1       Running   0          5s
       frontend-3823415956-dsvc5       1/1       Running   0          54m
       frontend-3823415956-k22zn       1/1       Running   0          54m
       frontend-3823415956-w9gbt       1/1       Running   0          54m
       frontend-3823415956-x2pld       1/1       Running   0          5s
       redis-master-1068406935-3lswp   1/1       Running   0          56m
       redis-slave-2005841000-fpvqc    1/1       Running   0          55m
       redis-slave-2005841000-phfv9    1/1       Running   0          55m

3. Run the following command to scale down the number of frontend Pods:

       kubectl scale deployment frontend --replicas=2

4. Query the list of Pods to verify the number of frontend Pods running:

       kubectl get pods

   The response should look similar to this:

       NAME                            READY     STATUS    RESTARTS   AGE
       frontend-3823415956-k22zn       1/1       Running   0          1h
       frontend-3823415956-w9gbt       1/1       Running   0          1h
       redis-master-1068406935-3lswp   1/1       Running   0          1h
       redis-slave-2005841000-fpvqc    1/1       Running   0          1h
       redis-slave-2005841000-phfv9    1/1       Running   0          1h
        
{% endcapture %}

{% capture cleanup %}
Deleting the Deployments and Services also deletes any running Pods. Use labels to delete multiple resources with one command.

1. Run the following commands to delete all Pods, Deployments, and Services.

       kubectl delete deployment -l app=redis
       kubectl delete service -l app=redis
       kubectl delete deployment -l app=guestbook
       kubectl delete service -l app=guestbook

   The responses should be:

       deployment "redis-master" deleted
       deployment "redis-slave" deleted
       service "redis-master" deleted
       service "redis-slave" deleted
       deployment "frontend" deleted    
       service "frontend" deleted
       
2. Query the list of Pods to verify that no Pods are running:

       kubectl get pods
        
   The response should be this: 

       No resources found.
        
{% endcapture %}

{% capture whatsnext %}
* Complete the [Kubernetes Basics](/docs/tutorials/kubernetes-basics/) Interactive Tutorials
* Use Kubernetes to create a blog using [Persistant Volumes for MySQL and Wordpress](/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/#visit-your-new-wordpress-blog) 
* Read more about [connecting applications](/docs/concepts/services-networking/connect-applications-service/)
* Read more about [Managing Resources](/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively)
{% endcapture %}

{% include templates/tutorial.md %}
