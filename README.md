# CLD - LAB05 : Kubernetes
**Group R : A. David, T. Van Hove**

**Date : 25.05.2023**

**Teacher : Prof. Marcel Graf**

**Assistant : Rémi Poulard**

In the first part of this lab we will install a Kubernetes test cluster on our local machine, and deploy a "To Do" reminder application. This application will et complete three-tier application (Frontend, API Server and Redis) using Pods. In the second part, we will deploy the same application on GKE (Google Kubernetes Engine). Finally,we will make the application resilient to failures and deploy it to the Kubernetes cluster of the school.

> In this document, the objectives of each chapter, the questions that are asked to us and the deliverables are identified by a quote

### Table of content

[toc]

# Task 1 : Deploy the application on a local test cluster

> In this task you will set up a local Kubernetes test cluster and deploy an example application to it.
>
> The goal of the provided example application is to store and edit a To Do list. The application consists of a simple Web UI (Frontend) that uses a REST API (API service) to access a Key-Value storage service (Redis). Ports to be used for Pods and Services are shown in the figure.
>



## 1.1 & 1.2- Installation of Minikube & Kubectl

We had no problem installing Minikube and kubectl. Here is the screenshot showing the installed version :

![](.\figures\minikube_installation.png)

## 1.3 - Create a one-node cluster on your local machine

The cluster creation process was easy to follow, and we did not have any issue doing it. The following screenshots shows the `minikube start` command and the cluster information, once it has been created.

![](figures/Start_cluster.png)

![](figures/cluster_info.png)

## 1.4 - Deploy the application

Once again, we didn't encounter any issue deploying the application. 

### Kubernetes description

The following screenshot shows the Kubernetes service description:

![](figures/Description_kubernetes-svc.png)

### Deploy the Redis Service and Pod

The following screenshot shows the Redis deployment with the `redis-svc` and `redis-pod` with the config files:

![](figures/redisSVC_redisPOD_creation.png)

The following screenshots show the description of the redis service and pod:

![](figures/Description_redis-svc.png)

![](figures/Description_redis-pod.png)

### Deploy the ToDo-API Service and Pod

We created the api-svc config file as asked in the lab:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    component: api
  name: api-svc
spec:
  ports:
  - port: 8081
    targetPort: 8081
    name: api
  selector:
    app: todo
    component: api
  type: ClusterIP
```

The following screenshot shows the deployment of the `api-svc` and `api-pod` with the config files. We can see that the service exposes the port 8081.

![](figures/api_Creation.png)

The following screenshot shows the description of the api service and pod

![](figures/Description_api-svc.png)

![](figures/api_description.png)

### Deploy the Frontend Pod

Here is our frontend-api configuration file. The `API_ENDPOINT_URL` environment variable should be set to the address of our API service within the Kubernetes cluster, so the URL would be `http://api-svc:8081`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    component: frontend
    app: todo
spec:
  containers:
  - name: frontend
    image: icclabcna/ccp2-k8s-todo-frontend
    ports:
    - containerPort: 8080
    env:
    - name: API_ENDPOINT_URL
      value: "http://api-svc:8081"
```

Now we just have to deploy the frontend pod:

![](figures/frontend_creation.png)

The following screenshot shows the description of the frontend pod :

![](figures/Description_frontend-pod.png)

### Verify the ToDo application

Then, using the kubectl port forwarding `kubectl port-forward frontend 8081:8080`, we can access the web app and see that it is served properly:

![](figures/todo_webApp.png)



# Task 2 - Deploy the application in Kubernetes engine

> In this task you will deploy the application in the public cloud service Google Kubernetes Engine (GKE).

## 2.1 - Create Project & 2.2 Create a cluster

> Take a screenshot of the cluster details from the GKE console.

We did not have any issue creating the cluster in GKE. Once created, GKE details page looked like this:

![](figures/GKE_Overview.png)

## 2.3 - Deploy the application on the cluster

We did not encounter any problems deploying the pods and services on the GKE cluster

## 2.4 - Deploy the ToDo-Frontend Service

First, we created the frontend-svc.yaml configuration file:

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    component: frontend
  name: frontend-svc
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: todo
    component: frontend
```



Then, in order to deploy the cluster, we had to install the gcloud auth plugin with this command `gcloud components install gke-gcloud-auth-plugin`. 



> Copy the output of the `kubectl describe` command to describe your load balancer once completely initialized.
>
> Find out the public URL of the Frontend Service load balancer using `kubectl describe`.

Once we deploy the service with de command `kubectl create -f frontend-svc.yaml` we can get the load balancer ip to access the todo app using the command `kubectl describe service frontend-svc`. 

![](figures/get-svc.png)

or for less details, with de command `kubectl get frontend-svc` and retrieve the EXTERNAL-IP. 

![](figures/get-svc-frontend-svc.png)

> Access the public URL of the Service with a browser. You should be able to access the complete application and create a new ToDo.

Finally we can access the deployed Todos app

![](figures/App_deployed_on_GKE.png)

> Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report (if they are unchanged from the previous task just say so).

Apart from the ip addresses, identifiers or dates, the other objects were not modified. We just added the front-end service. We provided the screenshot of the front-end service description earlier.

# Task 3 - Add and exercise resilience

> By now you should have understood the general principle of configuring, running and accessing applications in Kubernetes. However, the above application has no support for resilience. If a container (resp. Pod) dies, it stops working. Next, we add some resilience to the application.

## 3.1 Add deployments

Firstly we had to delete existing pods with the commands

````powershell
kubectl delete pod redis
kubectl delete pod api
kubectl delete pod frontend
````

Then we verified that our pods were actually deleted with the command  `kubectl get pods`

Then we had to create the 3 deployment configurations as follow:

redis-deployment.yaml:

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  labels:
    app: todo
    component: redis
spec:
  replicas: 1
  selector:
    matchLabels: # Used to determine which Pods are managed by this Deployment.
      app: todo
      component: redis
  template:
    metadata:
      labels:
        app: todo
        component: redis
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 6379
        args:
        - redis-server # Command to start the Redis server
        - --requirepass ccp2 #  Configures Redis to require a password 
        - --appendonly yes # Enables append-only mode to keep a log of all write operations
````

api-deployment.yaml:

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: todo
    component: api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo
      component: api
  template:
    metadata:
      labels:
        app: todo
        component: api
    spec:
      containers:
      - name: api
        image: icclabcna/ccp2-k8s-todo-api
        ports:
        - containerPort: 8081
        env:
        - name: REDIS_ENDPOINT
          value: redis-svc
        - name: REDIS_PWD
          value: ccp2
````

frontend-deployment.yaml:

````yaml
![Deployments](C:\Users\timot\Documents\HEIG\CLD\Labos\HEIG_CLD_Labo5\figures\Deployments.png)apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:
    app: todo
    component: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: todo
      component: frontend
  template:
    metadata:
      labels:
        app: todo
        component: frontend
    spec:
      containers:
      - name: frontend
        image: icclabcna/ccp2-k8s-todo-frontend
        ports:
        - containerPort: 8080
        env:
        - name: API_ENDPOINT_URL
          value: "http://api-svc:8081" # Internal URL of the API service within the cluster
````



> Use only 1 instance for the Redis-Server. Why?

If multiple instances wants to write in multiple databases, we need to set a synchronization to be sure that all the data written in any of the database are available to be read. But we don't want to manage database synchronization. For a so small application it would be overkill.

One the configuration files created, we only have to deploy them using `kubectl apply` command, and see if they are available :

![](figures/Deployments.png)

> Verify that the application is still working and the Replica Sets are in place.

redis deployment description:

![](figures/Description_redis-deployment.png)

api deployment description:

![](figures/Description_api-deployment.png)

frontend deployment description (note that we have taken this screenshot after adding a frontend instance):

![](figures/Description_frontend-deployment.png)

All resources (note that we have taken this screenshot 4 days after the effective deployment on GKE):

![](figures/task3_get_all.png)



## 3.2 Verify the functionality of replica set

In order to do that, we will monitor the pods and delete 2 pods to see what appends.

We can see in the screenshot below that when we delete a pod, it is automatically recreated after deletion. The api pod has been recreated in 3 seconds and the redis pod in 7 seconds.

![](figures/Task3_kill_pods.png)



> What happens if you delete a Frontend or API Pod?

1. We delete the Pod using `kubectl delete`.
2. The Kubernetes API server receives the deletion request and marks the Pod for deletion.
3. The kubelet on the node where the Pod was running notices that the Pod has been marked for deletion, stops the Pod's containers, and then removes the Pod from the node.
4. The Deployment or ReplicaSet controller in the Kubernetes control plane notices that the number of running Pods is less than the desired number of replicas specified in the Deployment or ReplicaSet.
5. The Deployment or ReplicaSet controller creates a new Pod to replace the deleted one. It submits a request to the Kubernetes API server to create the new Pod.
6. The Kubernetes scheduler assigns the new Pod to a node, and the kubelet on that node starts the Pod.
7. The new Pod starts, and the application becomes available again.



> What happens when you delete the Redis Pod?

Exactly the same as any other pod, except that the system might take a little more time to recreate the pod.



> How long does it take for the system to react?

The time it takes to recreate a Pod depends on multiple things. For example for the redis pod:

1. **Image size**: Larger images take longer to pull from the registry. If the Redis image is larger than the api image, or if the Redis image is not already cached on the node where the new Pod is being scheduled, it will take longer to pull the image, which could explain the delay.
2. **Startup time**: Some applications take longer to start up than others. Redis, for example, may take some time to initialize its data structures, load data into memory, or perform other setup tasks.
3. **Pod scheduling**: Kubernetes scheduler might take some time to schedule the pod depending on the current load, resource requests and limits.
4. **Persistent storage**: If the Redis Pod is configured with a Persistent Volume, the system might need additional time to detach the volume from the old Pod and attach it to the new Pod.



> How can you change the number of instances temporarily to 3?

It is possible to change temporarily the number of instances with the `kubectl scale` command, for example: `kubectl scale deployment frontend-deployment --replicas=3`

![](figures/Task3_add_temp_pod.png)

These changes are temporary and will be lost the next time we apply the original Deployment configuration with `kubectl apply`. 



> What autoscaling features are available? Which metrics are used?

Kubernetes provides the capability to automatically scale our application based on different metrics of the Horizontal Pod Autoscaler (HPA) and the Vertical Pod Autoscaler (VPA).

`HPA` automatically scales the number of Pods in a replication controller, deployment or replica set based on observed CPU utilization or, with custom metrics support, on some other application-provided metrics. The controller periodically adjusts the number of replicas in a Deployment or ReplicaSet to match the observed average CPU utilization to the target specified by the user. It can also scale based on memory usage and custom metrics (if custom metrics are set up in the cluster).

On the other hand, `VPA` automatically adjusts the CPU and memory reservations for the Pods to help "right size" the application. It continually analyzes the CPU and memory usage of Pods and dynamically adjusts their CPU and memory requests as necessary. This can be useful for applications that have changing resource requirements over time.

For `HPA`, the metrics used can be:

- **CPU Utilization**: This is the default metric. The target value is a percentage of the CPU request of the Pod containers.
- **Memory Utilization**: This is an optional metric and it has to be provided explicitly.
- **Custom Metrics**: Custom metrics are metrics not related to CPU and memory. These can be provided by the user or third-party services. Examples include request rate, response latency, etc.

`VPA` uses historical CPU and memory usage data to adjust the CPU and memory requests of the Pods.



> How can you update a component?

Updating a Deployment can be done by changing the Deployment configuration file and applying those changes with `kubectl apply`. The `kubectl apply` command updates the Deployment to match the state defined in the configuration file. Kubernetes will then automatically perform a rolling update to achieve the desired state. This means it will gradually replace old Pods with new ones, ensuring that the application remains available during the update.

If we're updating the image to deploy a new version of our application, we can also use the `kubectl set image` command to update the image directly without editing the configuration file. For example:

```bash
kubectl set image deployment/api-deployment api=icclabcna/ccp2-k8s-todo-api:v2
```

This command updates the `api` container in the `api-deployment` Deployment to use the `icclabcna/ccp2-k8s-todo-api:v2` image.

# Task 4 - Deploy on IICT Kubernetes cluster

> To show you that once you have your *YAML* file you can deploy  your app on any Kubernetes cluster without any changes, you will do the  same as task 3 but on the Kubernetes cluster of IICT. **You must be on VPN to access the IICT Kubernetes cluster, even in the school.**

## 4.1- setup kubectl

We had encountered no problems setting-up the kubectl configuration and connecting to the iict rancher website.

## 4.2 - Deploy the application

> Document your observations in the lab report. Document any difficulties you faced and how you overcame them. Copy the object descriptions into the lab report

For the deployment, we did exactly as we did for the GKE deployment, expect adding the namespace parameter in the commands. Here is a snapshot showing the deployment of the app and the status of the deployment:

![](figures/Task4_Deployment.png)

Here are the screenshots of the redis service and the redis deployment descriptions:

![](figures/Task4_description_redis_service.png)

![](figures/Task4_description_redis_deployment.png)

Here are the screenshots of the api service and the api deployment descriptions:

![](figures/Task4_description_api_service.png)

![](figures/Task4_description_api_deployment.png)

Here are the screenshots of the frontend service and the frontend deployment descriptions:

![](figures/Task4_description_frontend_service.png)

![](figures/Task4_description_frontend_deployment.png)



Finally, we could access the todos application using the ip address displayed in the frontend service description: `10.192.129.207`:
![](figures/App_deployed_on_IICT.png)
