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

### Connect App to Database
Run mongo-deploy.yml -> Run mongo-service.yml -> Run node-deploy.yml -> Run node-service.yml
These can be found in repository, database before app so it doesnt topple
Note: DB_HOST environmental variable is added to node deploy and uses mongo-service instead of an ip
Note: The app requires an older version of mongodb to run, the latest version 6 is incompatible, we have used 4.4

### PVC
![](https://i.imgur.com/UdDaEKr.png)

A Persistent Volume Claim (PVC) is a request for storage by a user. It is similar to a Pod in Kubernetes, as Pods consume node resources and PVCs consume PV resources.

* PVCs can request specific size and access modes (e.g., they can be mounted as read-write or read-only).
* A PVC is used by a Pod in its volume spec.
* Kubernetes connects a PVC to a suitable PV and makes the storage available to the Pod.
* If a PV was dynamically provisioned for a new PVC, the lifecycle of the PV is tied to that PVC.
* It allows Pods to use storage without being aware of the underlying storage infrastructure.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongo-db
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi
```

### HPA
The Horizontal Pod Autoscaler automatically scales the number of Pods in a replication controller, deployment, replica set, or stateful set based on observed CPU utilization or custom metrics support added in Kubernetes

* It ensures that a specific deployment or replica set has the necessary number of Pods to handle the load.
* It scales the number of Pods up or down based on CPU utilization or other select metrics.
* It uses the Metrics Server to fetch metrics like CPU utilization.
* It allows to set minReplicas and maxReplicas, which define the lower and upper limit for the number of Pods.

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler # hpa

metadata:
  name: sparta-mongo-db-deploy
  namespace: default

spec:
  maxReplicas: 9 # max number of pods
  minReplicas: 3 # min number of pods
  scaleTargetRef: # targets deployment
    apiVersion: apps/v1
    kind: Deployment
    name: mongo # node # nginx
  targetCPUUtilizationPercentage: 50 # 50% of cpu use
```
### Multiple Objects in Single YAML
```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  finalizers:
  - kubernetes.io/pv-protection
  labels:
    type: local
  name: mongodb-pv 
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
    type: ""
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem


# PVC for mongodb
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
 

# Deploy mongodb
---


apiVersion: apps/v1
kind: Deployment

metadata:
  name: mongodb-deployment
spec:
  selector:
    matchLabels:
      app: mongodb

  replicas: 3

  template:
    metadata:
      labels:
        app: mongodb

    spec:
      containers:
      - name: mongodb
        image: harrypaterson/tech241-mongodb:v1
        ports:
        - containerPort: 27017


# Mongodb service (SVC)

---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017



# Mongodb HPA autoscaling
---
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler #(hpa)

metadata:
  name: sparta-mongo-db-deploy
  namespace: default
  
spec:
  maxReplicas: 6 #(max nuber of instances/pods)
  minReplicas: 3 #(min nuber of instances/pods)
  scaleTargetRef: # Targets the node deployment
    apiVersion: apps/v1
    kind: Deployment
    name: mongodb
  targetCPUUtilizationPercentage: 50  # 50% of CPU use


# App PV
---
apiVersion: v1
kind: PersistentVolume
metadata:
  finalizers:
  - kubernetes.io/pv-protection
  labels:
    type: local
  name: app-pv 
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1Gi
  hostPath:
    path: /tmp/data
    type: ""
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem


# App PVC
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi


# App Deploy
---
apiVersion: apps/v1
kind: Deployment

metadata:
  name: app-deployment
spec:
  selector:
    matchLabels:
      app: app

  replicas: 3

  template:
    metadata:
      labels:
        app: app

    spec:
      containers:
      - name: app
        image: harrypaterson/tech241-app:v1
        ports:
        - containerPort: 3000

# App
---
apiVersion: v1
kind: Service

metadata:
  name: app-svc
  namespace: default

spec:
  ports:
  - nodePort: 30002
    port: 3000
    targetPort: 3000

  selector:
    app: app

  type: NodePort


# HPA
---
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler #(hpa)

metadata:
  name: sparta-app-deploy
  namespace: default
  
spec:
  maxReplicas: 6 #(max nuber of instances/pods)
  minReplicas: 3 #(min nuber of instances/pods)
  scaleTargetRef: # Targets the node deployment
    apiVersion: apps/v1
    kind: Deployment
    name: app
  targetCPUUtilizationPercentage: 50  # 50% of CPU use
```
# tech241_kubernetes
