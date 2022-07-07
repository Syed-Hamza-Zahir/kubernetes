# Kubernetes

## Kubernetes Architecture
- Kubernetes is an orchestrator for microservice apps

- Cluster - made up of one or more masters and a bunch of nodes
  - Nodes (each node is a VM or physical machine) (can have many nodes in a cluster)
    - Pods (can have 1 or more containers in a pod) -- Pods are the smallest deployable units of computing that can be created and managed in Kubernetes.
    - service (stable network endpoint to connect to one or multiple pods)

### Nodes - The Kubernetes Workers
- ``Kubelet`` - The main kubernetes agent
  - registers node with cluster
  - Watches apiserver
  - Instantiates pods
  - Reports back to master
- ``container engine`` - usually docker
- ``kube-proxy`` - handles kubernetes networking
  - pod IP address
    - All containers in a pods share a single IP
  - Load balances across all pods in a service

[!node.png](./node.png)

### Pods
- A pod is a ring-fenced (sandbox) environment to run containers
- containers always run inside of pods in Kubernetes
- All containers in a pod share the pod environment
  - eg share same IP, same volume, share memory

- ``Inter-pod communication:``
  - Every pod gets its own IP an so every pod can talk directly to every other pod
- ``Intra-pod communication``
  - Multiple containers in the same pod communicate over the shared localhost interface
  - To be reached from outside the pod they can be exposed on different ports

#### Pods and Scaling
- To scale, create more pods
  - Do NOT add more containers to a pod

### Service
- A service is a kubernetes object like a pod
  - It's an abstraction which defines a logical set of Pods and a policy by which to access them
  - It's an abstract way to expose an application running on a set of Pods as a network service.
- It sits in front of the backend and provides a dns name for backend pods
- A service is a k8s object and basically a firewall rule. This rule is created basically for two reasons.
  - To exposes pods to the external world.
  - To load balance between a set of pods and get rid of pod ip change if it dies.

### Publishing Services (ServiceTypes)
- ``ClusterIP``: Makes pods available on a stable IP but only from within the cluster
  - This is the default ServiceType
- ``NodePort``: Exposes the app from outside of the cluster
  - A ClusterIP service, to which the NodePort service will route, is automatically created.
- ``LoadBalancer``: Exposes the Service externally using a cloud provider’s load balancer.
  - NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created.

 ````
---

apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx
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
          image: smhzaihr/eng114_syed_docker
          ports:
            - containerPort: 80
          imagePullPolicy: Always 
 ````
- ``labels``: Labels are key/value pairs that are attached to objects (in this case the service). Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users
- ``selector``: Allows users to filter a list of resources based on labels. The set of Pods targeted by a Service is usually determined by a selector.
- ``port: 80``: used to define the port on which the service listens on
- ``targetPort: 80``: the actual port on which your application is running inside the container
- Once the service receives traffic from an external source:
  - it sends the traffic received on ``nodePort`` and forwards that to the ``port`` the service is listening on
  - it redirects the traffic received on ``port`` to ``targetPort`` which is the directive used to define port on which container has exposed the application.


- ``kubectl expose deployment <deployment_name> --port 80 --type LoadBalancer``


### Deployments
- A Deployment provides declarative updates for Pods and ReplicaSets.
- Deployed via YAML (or JSON) manifests
- Deployed via the apiserver
- Deployments are used for:
  - Rolling updates and rollbacks
  - also deploying pods (replica sets - scalability and reliability)
- ``kubectl apply -f deploy.yml --record``

````
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  ports:
   - nodePort: 30442 
     protocol: TCP
     port: 80
     targetPort: 80
     
  selector:
    app: nginx
  type: NodePort
````
- ``labels``: Assigns this label when creating the Deployment
- ``spec.selector.matchLabels``: means control the ReplicaSet/Pods which have this label
- ``template``: is a a podTemplate. It describes the pods that are launched.
- ``template.metadata.labels``: Assigns this label when creating the ReplicaSet/Pod



### Commands

- ``kubectl get nodes``
  - This command shows all nodes that can be used to host our applications.
- ``kubectl describe deploy <deployment_name>``
- ``kubectl apply -f ./pod.yml``
  - creates or applies a config to a resource
- ``kubectl apply -f deploy.yml --record``
  - use the ``--record`` flag for deployments for the deployment history
- ``kubectl expose deployment <deployment_name> --port 80 --type LoadBalancer``
   - Expose containers to the internet
   - Kubernetes created a service and an external load balancer with a public IP address attached to it. The IP address remains the same for the life of the service. Any network traffic to that public IP address is routed to pods behind the service
- ``kubectl logs <name_of_pod>``
  - get the logs of a pod

# Deploying Nodejs Sample App and MongoDB using Kubernetes


## MongoDB deployments and Internal LoadBalancer
- create the mongodb_deploy.yml and mongodb_service.yml files

- ``kubectl apply -f db_deploy.yml``
- ``kubectl apply -f svc_db.yml``

- Find the external IP of the mongodb service ``mongodb-svc``
  - run ``kubectl get svc``


## App Deployment and LoadBalancer
- create the app_deploy.yml and app_service.yml file

- replace ``mongodb-svc`` in ``value: "mongodb://mongodb-svc:27017/posts"`` with the external IP of the mongodb service ``mongodb-svc`` then
- ``kubectl apply -f app_deploy.yml``

- ``kubectl apply -f svc-app.yml``
