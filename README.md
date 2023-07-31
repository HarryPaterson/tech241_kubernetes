<h1 style="text-align: center;">Kubernetes</h1>

### Contents
* What is Kubernetes?
* Kubernetes Architecture
* Why should we learn and use it?
* Who is using Kubernetes?
* Benefits to business
* Kubernetes Objects
* Labels and Selectors
* Deployment and Services

### What is Kubernetes (K8s)?

Kubernetes (often abbreviated to K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. It groups containers that make up an application into logical units for easy management and discovery.

### Kubernetes Architecture
![](https://www.cncf.io/wp-content/uploads/2020/08/Kubernetes-architecture-diagram-1-1.png)

### Why Should We Learn and Use It?

- **Highly flexible**: Kubernetes can run anywhere, from your laptop, to VMs on a cloud provider, to a rack of bare metal servers.
- **Increased efficiency**: With Kubernetes, you can quickly and efficiently respond to customer demand: 
    - Deploy your applications quickly and predictably.
    - Scale your applications on the fly.
    - Roll out new features seamlessly.
    - Limit hardware usage to required resources only.

### Who Is Using Kubernetes?

- Spotify
- Huawei
- IBM
- Pinterest
- eBay
- Shopify
- Squarespace

### Benefits to Business

- **Agility and Speed**: With Kubernetes, businesses can rapidly respond to market changes.
- **Reduced Cost**: Kubernetes makes better use of hardware to maximize resources that are needed to run your enterprise applications.
- **Portability and Flexibility**: Run your applications anywhere without the need to modify the underlying code.
- **Improved Productivity**: Automates various manual processes; managing deployments and scaling is more efficient.

### Kubernetes Objects

- **Pods**: The smallest and simplest unit in the Kubernetes object model that you create or deploy.
- **Deployments**: A Deployment provides declarative updates for Pods and ReplicaSets.
- **Services**: An abstract way to expose an application running on a set of Pods as a network service.
- **ReplicaSets**: Ensures that a specified number of pod replicas are running at any given time.

### Concept of Labels and Selectors in Kubernetes

**Labels** are key/value pairs attached to objects and can be used to organize and to select subsets of objects. 

**Selectors** allow the user to filter keys based on the labels.

### Deployment and Services
We will be using: 
```
kubectl create -f file.yml
```
For both our deployments and services. We can view the status of each with
```
kubectl get deploy
```
or
```
kubectl get svc
```
Example of deploy file:
```
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: harrypaterson/tech241-nginx:v1
          ports:
          - containerPort: 80
```
Example of service file:
```
---
# Select the type of API version and type of service/object
apiVersion: v1
kind: Service
# Metadata for name
metadata:
  name: nginx-svc
  namespace: default # sre
# Specification to include ports Selector to connect to the deployment
spec:
  ports:
  - nodePort: 30001
    port: 80
    targetPort: 80
    # range is 30000-32768
    # Let's define the selector and label to connect to nginx deployment
  selector:
    app: nginx # This label connects this service to deployment
    #Creating the NodePort type of deployment
  type: NodePort
```
# tech241_kubernetes
