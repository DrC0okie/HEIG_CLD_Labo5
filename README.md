# CLD - LAB05 : Kubernetes
**Group R : A. David, T. Van Hove**

**Date : 25.05.2023**

**Teacher : Prof. Marcel Graf**

**Assistant : Rémi Poulard**

In this lab we will install a Kubernetes test cluster on your local. Then, we will deploy a rovided "To Do" reminder application. This application will et complete three-tier application (Frontend, API Server and Redis) using Pods.

In the second part of this lab, we will deploy the same application on a public cloud service who run Kebernetes (Google Kubernetes Engine).

Finally, in the third part of this lab, we will make the application resilient to failures.

### Table of content

[toc]

# Task 1 : Deploy the application on a local test cluster

In this task, we will set up a local Kubernetes test cluster and deploy an example application to it.

## Subtask 1.1 & 1.2- Installation of Minikube & Kubectl

We had no problem installing Minikube and kubectl. Proof of this the screenshot showing the installed version :

![](.\figures\minikube_installation.png)

## Subtask 1.3 - Create a one-node cluster on your local machine

The cluster creation process was easy to follow, and we did not have any issue doing it. The following screenshots shows the `minikube start` command and the cluster information, once it has been created.

![](figures/Start_cluster.png)

![](figures/cluster_info.png)

## Subtask 1.4 - Deploy the application

Once again, we didn't encounter any issue deploying the application. 

### Redis deployment

The following image shows the Redis deployment with the `redis-svc` and `redis-pod` with config files:

![](figures/redisSVC_redisPOD_creation.png)

The following screenshot shows the description of the redis service:

![](figures/Description_redis-svc.png)

### Api deployment

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

The following screenshot shows the description of the api pod

![](figures/api_description.png)

### Front-end deployment

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

Then, using the kubectl port forwarding `kubectl port-forward frontend 8081:8080`, we can access the web app and see that it is served properly:

![](figures/todo_webApp.png)

# Task 2 - Deploy the application in Kubernetes engine



## Subtask 2.1 - Create projet



## Subtask 2.2 - Create a cluster





## Subtask 2.3 - Deploy the application on the cluster



## Subtask 2.4 - Deploy the todo-frontend service





## Verify the todo application



# Task 3 - Add and exercise resilience







# Task 4 - Deploy on IICT Kubernetes cluster





