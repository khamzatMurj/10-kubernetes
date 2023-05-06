## Notes on the videos for Module 10 "Container Orchestration with Kubernetes"
<br />

<details>
<summary>Video: 1 - Introduction to Kubernetes</summary>
<br />

Kubernetes (aka K8s or Kube) is an open source container orchestration tool, developed by Google. It automates many processes involved in deploying, managing and scaling containerized applications.

It provides
- high availability or no downtime
- automatic scaling
- disaster recovery - Backup and Restore
- self-healing

</details>

*****

<details>
<summary>Video: 2 - Kubernetes Components</summary>
<br />

Main Kubernetes components worth knowing:
- Pod
- Service
- Ingress
- Config Map
- Secret
- Deployment
- Statefulset
- Volumes

### Pod
- Smallest unit of K8s
- An abstraction over container
- Usually 1 container per Pod (1 main container and n helper containers)
- Each Pod gets its own internal IP address (not each container)
- Pods are ephemeral (can die and be replaced)
- A new IP address is assigned on re-creation

### Service
- A permanent IP address that can be attached to a Pod
- Lifecycles of Service and Pod are not connected, so if a Pod crashes and gets re-created, the Service and its IP address will stay the same
- If Pods are replicated on multiple nodes, the Service can also serve as a loadbalancer for the Pods of the same type
- When creating a service you can specify its type:
  - Internal Service: By default, for example a database, which should not be accessible from outside
  - External Service: Application accessible through browser
- The address of an external Service is just `http://<node-ip>:<service-port>`

### Ingress
- Ingress is the entrypoint to your K8s cluster
- Request goes to Ingress first, which does the forwarding to the Service
- The address of the Ingress is `https://<my-app-domain>`

### ConfigMap & Secret
For external configuration, Kubernetes has these 2 components:
- ConfigMaps store non-confidential data in key-value pairs
- Secrets store sensitive data such as passwords or tokens. See [Managing Secrets](https://blog.aquasec.com/managing-kubernetes-secrets) for more information.
- Pods can consume ConfigMaps and Secrets 
  - as environment variables, 
  - CLI arguments 
  - or as config files in a Volume

### Volume
When a container crashes, K8s restarts the container but with a clean state. Meaning your data is lost!

- The Volume component attaches a physical storage on a hard drive to your Pod
- The storage could be either on a local server or outside the K8s cluster

Think of storage as an external hard drive plugged in to your K8s cluster. As a consequence, K8s doesn't manage any data persistence. So you are responsible for backing up, replicating the data etc.

### Deployment
A Deployment is a blueprint of Pods. By defining the number of replicas, K8s creates the Pods. The Service acts as a loadbalancer for the replicated Pods. Having load balanced replicas our setup is much more robust.

### StatefulSet
A StatefulSet is a blueprint for stateful applications, like databases etc. In addition to replicating features, StatefulSet makes sure database reads and writes are synchronized to avoid data inconsistencies.

### Layers of Abstraction
Deployment manages a\
-> ReplicaSet, which manages\
--> Pods, which are an abstraction of\
---> Containers

</details>

*****

<details>
<summary>Video: 3 - Kubernetes Architecture</summary>
<br />

A Kubernetes cluster consists of a set of machines, called "Nodes". There are two types of Nodes:
- **Worker Nodes** run the containerized applications. Each Node runs multiple Pods.
- **Control Planes** manage the Worker Nodes and their Pods in the cluster. Replicated over multiple machines.

### Worker Node
On each worker node 3 processes need to be installed:
- **Container Runtime:** responsible for running containers (e.g. containerd, CRI-O, Docker)
- **Kubelet:** Agent that makes sure containers are running in a Pod. Talks to underlying server (to get resources for Pod) and container runtime (to start containers in Pod)
- **Kube-Proxy:** A network proxy with intelligent forwarding of requests to the Pods (e.g. forwarding to a Pod running on the same Node to avoid network traffic)

### Control Plane
Control Planes makes global decisions about the cluster. They detect and respond to cluster events. On each control plane 4 processes need to be installed:
- **API server:** The cluster gateway - single entrypoint to the cluster. Acts as a gatekeeper for authentication, validating the request. Clients to interact with the API server are UI, API or CLI (kubectl).
- **Scheduler:** Decides on which Node a new Pod should be scheduled. Factors taken into account for scheduling decisions are resource requirements, hardware/software/ policy constraints, data locality, ... After having chosen the node, the Kubelet on that node does the actual work of running the Pod.
- **Controller Manager:** Detects state changes, like crashing of Pods, and tries to recover the cluster state as soon as possible. For that it makes request to the Scheduler to re-schedule those Pods.
- **etcd:** K8s' backing store for all cluster data. A consistent, high-available key-value store. Every change in the cluster gets saved or updated into it. All other processes like Scheduler, Controller Manager etc. do their work based on the data in etcd as well as communicate with each other through etcd store.

### Increase Cluster Capacity
To add more control plane nodes or worker nodes to the cluster, just get a fresh machine, install the required K8s processes on it and join it to the K8s cluster using a K8s command.

</details>

*****

<details>
<summary>Video: 4 - Minikube: Setting up a Local Kubernetes Cluster</summary>
<br />

### Minikube
Minikube implements a local K8s cluster. This is useful for local K8s application development. Control Plane and Worker processes run on one machine. You can run Minikube either as a container or virtual machine on your laptop.

#### Install Minikube (on Mac)
- [Installation Guide for Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Installation Guide for Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)

```sh
# using docker
brew update
brew install minikube
minikube start --driver=docker
minikube status

# using hyperkit
brew update
brew install hyperkit
brew install minikube
minikube start --vm-driver=hyperkit
minikube status

# kubectl has been installed as a dependency of minikube
# we don't have to install it separately
kubectl get nodes
```

</details>

*****

<details>
<summary>Video: 5 - Main kubectl commands</summary>
<br />

Kubectl is a CLI tool to interact with your K8s cluster. In order for kubectl to access a K8s cluster, it needs a kubeconfig file, which is created automatically when deploying your minikube cluster. By default, the config file is located at `~/.kube/config`. This location can be overridden by exporting an environment variable KUBECONFIG.

### Basic kubectl Commands

See [reference](https://kubernetes.io/docs/reference/kubectl/)

```sh
# list components
kubectl get all
kubectl get node(s)
kubectl get pod(s)
kubectl get service(s)
kubectl get deployment(s)
kubectl get replicaset(s)

# create components
kubectl create {k8s component} {name} {options}
kubectl create deployment nginx-depl --image=nginx

# edit / delete components
kubectl edit {k8s component} {name}
kubectl delete {k8s component} {name}
kubectl delete -f config-file.yaml

# debug pods
kubectl logs {pod-name}
kubectl describe {pod-name}

# enter the container
kubectl exec -it {pod-name} -- bash

# apply a configuration file
kubectl apply -f config-file.yaml

# create a configuration file
kubectl create deployment --image=nginx helloworld -o yaml --dry-run=client > helloworld-deployment.yaml

# export a configuration file
kubectl get deployment helloworld -o yaml > helloworld-deployment-orig.yaml 

# --- get help ---

kubectl options
kubectl help
kubectl create --help
kubectl create deployment --help

# metrics
kubectl top # returns current CPU and memory usage for a cluster’s pods or nodes, or for a particular pod or node if specified
```

</details>

*****

<details>
<summary>Video: 6 - YAML Configuration File</summary>
<br />

Kubernetes configuration/manifest files are declarative, i.e. they specify the desired state of a K8s component. Each configuration file has 3 parts:
- **metadata**
- **specification:** the attributes of "spec" are specific to the component kind
- **status:** automatically generated and added by K8s; K8s gets this information from etcd, which holds the current/actual state of any K8s component; if the actual state differs from the specified/desired state, K8s tries to fix that and reach the desired state

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devops-deployment
  labels:
    app: devops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: devops
  template: # Pod configuration
    metadata:
      labels:
        app: devops
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 8080
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2023-04-16T16:33:13Z"
    lastUpdateTime: "2023-04-16T16:33:13Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2023-04-16T16:33:02Z"
    lastUpdateTime: "2023-04-16T16:33:13Z"
    message: ReplicaSet "tododevops-deployment-74b5b8bb9f" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  updatedReplicas: 1
```

The configuration of a Deployment is a bit special since it's an abstraction over Pod. Inside the Deployment spec we have the Pod configuration (own "metadata" and "spec" section = blueprint for Pod).

### Connecting components (Labels, Selectors & Ports)
Labels are key/value pairs that are attached to resources. The key and value can be randomly chosen. Via a selector the user can identify a set of resources (since labels do not provide uniqueness).

Connecting Deployment to Pods: The label of the Pod (in the template's metadata) is matched by the selector (matchedLabels) in the deployment spec.

Connecting Services to Deployments / Pods: The label of the Deployment and Pod is matched by the selector in the Service spec (the service must know which Pod belongs to it):

```yaml
apiVersion: v1
kind: Service
metadata: 
  name: devops-service
spec:
  selector:
    app: devops
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```

The `port` of a service specifies the port the service is listening on, whereas the `targetPort` defines the port the service is forwarding requests to. The `targetPort` of the service must match the `containerPort` of the Pod.

To see if a service is connected with the right pods, execute `kubectl describe service <service-name>` to see the endpoints of the service. The IP address of a Pod can be displayed with `kubectl get pod <pod-name> -o wide`.

To get the whole configuration file of a running deployment (and check the status information added by K8s), execute `kubectl get deployment <deployment-name> -o yaml > deployment-config-result.yaml` or `kubectl get deployment <deployment-name> -o yaml | less`.

</details>

*****

<details>
<summary>Video: 7 - Complete Demo Project (Deploying Application in K8s Cluster)</summary>
<br />

Overview:
- User updates entries in database via browser
- External service for Mongo Express as UI
- Internal service for MongoDB as database
- ConfigMap and Secret holds the MongoDB's endpoint (Service name of MongoDB) and credentials (user, pwd), which gets injected to MongoExpress Pod, so MongoExpress can connect to the DB

K8s Components needed in this setup:
- 2 Deployments / Pods
- 2 Services (1 internal, 1 external)
- 1 ConfigMap (holding internal service name needed by Mongo Express to connect to Mongo DB)
- 1 Secret (holding DB credentials needed by Mongo DB and Mongo Express)

Precondition: Minikube cluster is running on local machine (inside a Docker container).

### MongoDB Deployment
Create a file called `mongodb.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb-deployment
  labels:
    app: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
```

We don't add the username and password as plaintext values to this file. Instead we create a secret holding these values and reference it.

### Secret
Create a file called `mongodb-secret.yaml` with the following content:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: bW9uZ28=
  mongo-root-password: c2VjcmV0
```

The username and password strings are base64 encoded plain-text values. To get them enter the following commands in a shell:
```sh
echo -n 'mongo' | base64 # -n: suppresses the trailing newline character
# => bW9uZ28=
echo -n 'secret' | base64
# => c2VjcmV0
```

Because we reference the secret from within the deployment, the secret must exist when we create the deployment. So we create the secret first and then the deployment:
```sh
kubectl apply -f mongodb-secret.yaml
kubectl apply -f mongodb.yaml

# wait until the pod is ready
kubectl get pods --watch
#NAME                                  READY   STATUS              RESTARTS   AGE
#mongodb-deployment-5d966bd9d6-2ng9f   0/1     ContainerCreating   0          5s
#mongodb-deployment-5d966bd9d6-2ng9f   1/1     Running             0          55s
```

### Internal Service for MongoDB Pod
Append the following content to the `mongodb.yaml` file:
```yaml
--- # delimiter for multiple documents within one configuration yaml
apiVersion: v1
kind: Service
metadata: 
  name: mongodb-service
spec:
  selector:
    app: mongodb # must match the label of the pod
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017 # must match the containerPort of the pod
```

Now re-apply the changes: `kubectl apply -f mongodb.yaml`.

### MongoExpress Deployment
Create a file called `mongoexpress.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express-deployment
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_SERVER
          valueFrom:
            configMapKeyRef:
              name: mongodb-configmap
              key: database_url
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: mongo-root-password
```

### ConfigMap
Create a file called `mongodb-configmap.yaml` with the following content:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service # must match the name of the internal service
```

Just like with the secret before, the config-map must exist when we create the deployment. So we create the config-map first and then the deployment:
```sh
kubectl apply -f mongodb-configmap.yaml
kubectl apply -f mongoexpress.yaml

# wait until the deployment is ready
kubectl get deployments --watch
#NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
#mongo-express-deployment   0/1     1            0           1s
#mongodb-deployment         1/1     1            1           82m
#mongo-express-deployment   1/1     1            1           43s
```

### External Service for MongoExpress Pod
Append the following content to the `mongoexpress.yaml` file:
```yaml
---
apiVersion: v1
kind: Service
metadata: 
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  type: LoadBalancer # badly chosen name, because internal services also do load-balancing
                     # just assigns the service an external IP address
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000 # must be between 30'000 and 32'767
```

Now re-apply the changes and list all services:
```sh
kubectl apply -f mongoexpress.yaml
kubectl get services
# =>
#NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
#kubernetes              ClusterIP      10.96.0.1      <none>        443/TCP          42h
#mongo-express-service   LoadBalancer   10.97.135.6    <pending>     8081:30000/TCP   15s
#mongodb-service         ClusterIP      10.108.21.76   <none>        27017/TCP        17m
```

In "normal" K8s clusters the LoadBalancer service would get an external IP address. Because we are in a minikube cluster, the external IP address just shows `<pending>`. To assign an IP address accessible by our local browser we execute `minikube service mongo-express-service` which assigns an IP address and opens the browser pointing to this IP address and the external port of the specified service (or in the case of minikube running in a docker container, it opens a tunnel into the docker container):

```sh
minikube service mongo-express-service
#|-----------|-----------------------|-------------|---------------------------|
#| NAMESPACE |         NAME          | TARGET PORT |            URL            |
#|-----------|-----------------------|-------------|---------------------------|
#| default   | mongo-express-service |        8081 | http://192.168.49.2:30000 |
#|-----------|-----------------------|-------------|---------------------------|
#🏃  Start Tunnel für den Service mongo-express-service
#|-----------|-----------------------|-------------|------------------------|
#| NAMESPACE |         NAME          | TARGET PORT |          URL           |
#|-----------|-----------------------|-------------|------------------------|
#| default   | mongo-express-service |             | http://127.0.0.1:59516 |
#|-----------|-----------------------|-------------|------------------------|
#🎉  Öffne Service default/mongo-express-service im Default-Browser...
#❗  Weil Sie einen Docker Treiber auf darwin verwenden, muss das Terminal während des Ausführens #offen bleiben.
```

</details>

*****

<details>
<summary>Video: 8 - Namespaces (Organizing Components)</summary>
<br />

Namespaces organise resources in a K8s cluster. There are 4 namespaces available in a new cluster:
```sh
kubectl get namespaces
# =>
# NAME              STATUS   AGE
# default           Active   46h
# kube-node-lease   Active   46h
# kube-public       Active   46h
# kube-system       Active   46h
```

- kube-system: don't modify anything in this namespace; system processes and control-plane processes are running in this namespace
- kube-public: contains publicly accessible data (`kubectl cluster-info`)
- kube-node-lease: holds information about the heartbeats of nodes; each node has its associated lease object whithin this namespace (holding availabilty information)
- default: start deploying your application in the default namespace

To create a new namespace, use the command
```sh
kubectl create namespace <ns-name>
```

Or apply a config file of the following form:
```sh
apiVersion: v1
kind: Namespace
metadata:
  name: <ns-name>
```

Within a configuration file the target namespace for the component can be specified in the metadata section:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
  namespace: my-namespace
data:
  database_url: mongodb-service
```

When applying a configuration file using `kubectl` the target namespace can be specified using the option --namespace or short -n:
```sh
kubectl apply -f config.yaml --namespace=my-namespace
```

In `kubectl` commands listing components you can add the option --namespace or short -n to specify the namespace from which you want to display the components:
```sh
kubectl get deployments -n=dev
```

### Use Cases for When to Use Namespaces
- Group resources logically (e.g. database, monitoring, etc.)
- Isolate team resources to avoid conflicts
- Define environments and share resources between them (e.g. dev, stage, prod, using resources in one elastic-stack namespace; or prod-blue, prod-green using resources in one elastic-stack namespace)
- Limit permissions and compute resources (CPU, RAM, Storage) per namespace

Restrictions: In namespace B you cannot reference a ConfigMap or a Secret defined in namesapce A.

But you can share a Service defined in namespace A and use it in namespace B too. For example if the service 'mongodb-service is defined in the namespace 'database' and you want to reference it from a ConfigMap specification in namespace 'my-namespace', you just add '.database' to the service-name:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
  namespace: my-namespace
data:
  database_url: mongodb-service.database
```

There are components that cannot be added to a namespace, but live globally in the cluster (like PersistentVolume, Node or Namespace itself). To list all resources that cannot be added to a namespace, execute
```sh
kubectl api-resources --namespaced=false
```

</details>

*****

<details>
<summary>Video: 9 - Services (Connecting to Applications inside Cluster)</summary>
<br />

Each Pod has its own IP address, but when a Pod is replaced by a new one (e.g. because the old one crashed), it gets a new IP address.

Services belong to a Pod (or a group of replicated Pods) and have a lifecycle independent from the Pods. They have a stable IP address and act as a load-balancer for the replicated Pods.

There are three different types of Services:
- ClusterIP
- NodePort
- LoadBalancer

### ClusterIP Services

This is the default type of services (if no type is specified). It is an internal service, not accessible from outside the cluster. But all Pods in the cluster can talk to this internal service.

How does a Service know which Pods belong to it? The set of Pods targeted by a Service is determined by a selector. E.g. the selector `app: mongodb` (under the Service's spec > selector attribute) selects any Pod with the label `app: mongodb` (under the Pod's metadata > labels attribute). Selectors may also have multiple key value pairs. Then the Pods must have all these labels to get selected.

A Pod may have multiple containers (a main container and its helper containers / side-cars). Each container gets its own port within the Pod. How does the Service know which container/port the request must be forwarded to? The attribute `targetPort` (under the Service's spec > ports attribute) matches the port of the container (under the Pod's spec > containers > ports attribute).

#### Subtype Multi-Port Internal Service
If a helper-container within a port must be accessible too (e.g. a mongodb-exporter used by a monitoring component), the Service for that Pod must have multiple ports:
```yaml
apiVersion: v1
kind: Service
metadata: 
  name: mongodb-service
spec:
  selector:
    app: mongodb
  ports:
    - name: mongodb
      protocol: TCP
      port: 27017
      targetPort: 27017
    - name: mongodb-exporter
      protocol: TCP
      port: 9216
      targetPort: 9216
```

Note that port definitions must be given a name as soon as you have more than one.

#### Subtype Headless Internal Service

When a client needs to communicate with one specific Pod directly, instead of a randomly (by the service) selected one. Use case: when Pod replicas are not identical, for example with stateful apps, like when only master is allowed to write to database.

When a client needs to get the IP address of a Service, it can do a K8s DNS lookup and get the cluster IP address of the Service. But if you set the `clusterIP` attribute of a Service to be `None`, that Service won't get an IP address and the K8s DNS lookup will directly return the IP addresses of the Pods behind that Service.

```yaml
apiVersion: v1
kind: Service
metadata: 
  name: mongodb-service-headless
spec:
  clusterIP: None # <---
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

### NodePort Services
NodePort Services are external services, i.e. they are directly accessible from outside the cluster. They are exposed on each node's IP at a static port. So the URL is the IP address of Worker Node plus the node port.

```yaml
apiVersion: v1
kind: Service
metadata: 
  name: mongodb-service-nodeport
spec:
  type: NodePort
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
      nodePort: 30000 # must be within the range of 30000 .. 32767
```

Note that a ClusterIP Service, to which the NodePort Service routes, is automatically created (using the `port` and `targetPort` attributes).

If the Pods related to a NodePort Service are living on different Nodes, the NodePort Service will act as a load-balancer over these Nodes (same as load-balancing on Pod level, but on Node level).

### LoadBalancer Services
Open up ports on Nodes and making them directly accessible from outside the cluster is not secure as external traffic now has access to fixed ports on each Node.

A better solution is to use a Service of type `LoadBalancer`. It exposes the Service externally using the cloud provider's load balancer. NodePort and ClusterIP Services, to which the external load balancer routes, are automatically created. The load balancer will be the only entry point to the services.

```yaml
apiVersion: v1
kind: Service
metadata: 
  name: mongodb-service-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
      nodePort: 30000 # must be within the range of 30000 .. 32767
```

In production you will either configure Ingress or a LoadBalancer service to make a Service accessible from outside the cluster.

</details>

*****

<details>
<summary>Video: 10 - Ingress (Connecting to Applications outside Cluster)</summary>
<br />

Instead of accessing your application via `http://<external-service-ip>:port` you would prefer something like `https://myapp.com/`. This is where Ingress comes in. It acts as the entry-point of your application and pass incoming requests over to an internal Service.

### Configuration YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.com # map domain name to IP address of the node, which is the entrypoint
    http: # 2nd step: incoming request gets forwarded to internal service
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-internal-service
            port:
              number: 8080
```

### Ingress Controller
You need an implementation for Ingress, which is the Ingress Controller (a Pod). It
- evaluates all the rules defined in Ingress configuration files,
- manages redirections,
- is the entry-point to the cluster.

There are many third-party [implementations](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/). The implementation of K8s is the [Nginx Ingress Controller](https://www.nginx.com/products/nginx-ingress-controller/).

In production environment you usually have load-balancer or another kind of proxy which acts as the only entry point to your k8s cluster and redirects incoming requests to the Ingress Controller.

### Install Ingress in Minikube
To install an Ingress Controller in Minikube, you can automatically start the K8s Nginx implementation of Ingress Controller by executing the following command:
```sh
minikube addons enable ingress
# Nachdem das Addon aktiviert wurde, führen Sie bitte "minikube tunnel" aus, dann sind ihre Resourcen über "127.0.0.1" erreichbar

# check
kubectl get pods -n kube-system
```

### Configure Ingress for Access to K8s Dashboard
Start K8s Dashboard in Minikube:
```sh
minikube dashboard # this opens the K8s dashboard in the default browser
```

Collect the information needed to configure Ingress:
```sh
# in a new terminal
kubectl get namespaces
# NAME                   STATUS   AGE
# default                Active   4d22h
# ingress-nginx          Active   23h
# kube-node-lease        Active   4d22h
# kube-public            Active   4d22h
# kube-system            Active   4d22h
# kubernetes-dashboard   Active   22h   <----

kubectl get all -n kubernetes-dashboard
# ...
# NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
# service/kubernetes-dashboard        ClusterIP   10.110.201.35   <none>        80/TCP     23h
# ...
```

Create a file called `dashboard-ingress.yaml` with the following content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kubernetes-dashboard
spec:
  rules:
  - host: dashboard.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port: 80
```

Apply this config file and wait for an IP address to be assigned:
```sh
kubectl apply -f dashboard-ingress.yaml

kubectl get ingress -n kubernetes-dashboard --watch
# NAME                CLASS   HOSTS           ADDRESS        PORTS   AGE
# dashboard-ingress   nginx   dashboard.com                  80      5s
# dashboard-ingress   nginx   dashboard.com   192.168.49.2   80      32s
```

In order to map the defined host `dashboard.com` to the IP address assigned to ingress, we add an entry `192.168.49.2    dashboard.com` to the hosts file `/etc/hosts`. Now we can access the dashboard using `http://dashboard.com`.

****
Note: This seems not to work when using minikube with Docker driver.
****

### Define Multiple Paths for the Same Host
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /analytics
        pathType: Prefix
        backend:
          service:
            name: analytics-service
            port: 3000
      - path: /shopping
        pathType: Prefix
        backend:
          service:
            name: shopping-service
            port: 8080
```

### Define Multiple Subdomains
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: analytics.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: analytics-service
            port: 3000
  - host: shopping.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: shopping-service
            port: 8080
```

### Configuring TLS Certificate
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-secret-tls
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: myapp-service
            port: 3000
---
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret-tls
  namespace: default
data:
  tls.crt: <base64 encoded cert>
  tls.key: <base64 encoded key>
type: kubernetes.io/tls # <-----
```

</details>

*****

<details>
<summary>Video: 11 - Volumes - Persisting Application Data</summary>
<br />

K8s offers no data persistence out of the box.

A volume is a directory (with some data in it), which is accessible to the containers in a Pod. Persistent volumes exist beyond the lifetime of a Pod, even beyond the lifetime of the cluster. Volumes must be available on all nodes, because it is not known on which node a new pod gets started when the old one is replaced.

 The way to persist data in K8s using volumes is with these 3 resources:
- Persistent Volume (PV): Storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes
- Persitent Volume Claim (PVC): A request for storage by a user; similar to Pods, while Pods consume node resources, PVCs consume PV resources
- Storage Class (SC): SC provisions PV dynamically when PVC claims it

### Persitent Volume
Persistent Volumes are NOT namespaced, so PV resource is accessible to the whole cluster. Depending on storage type, spec attributes differ. In official documentation you can find a complete list of more than 25 storage backends supported by K8s.

PV is configured in a yaml file (kind: `PersistentVolume`). The spec attribute differs depending on the storage type. The following is an example of a Persitent Volume using an NFS server:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

Persistant Volume is just an abstraction. The real physical storage resource (e.g. local filesystem, remote NFS, cloud storage) has to be made available to the cluster and provisioned by a system administrator. It can be seen as a plugin to the cluster.

Local volumes are tied to one specific node. And they don't survive a cluster crash. For database storage you should always use a remote volume type.

### Persistent Volume Claim
System administrators provision the persistent volume resources for a cluster. And K8s users (developers) have to make these volumes available for their Pods. They can claim volume resources using Persistent Volume Claims (PVC). PVCs are also createed using a yaml configuration. Here's an example:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
```

Based on the specified requirements of the PVC, the best matching persistent volume resource is chosen. The volume claimed in the PVC can then be made available to the Pod by referencing the PVC:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    name: nginx
    image: nginx
    volumeMounts: # mounting the volume into the container
    - name: my-pod-dir
      mountPath: "/var/www/html"
  volumes: # mounting the volume into the Pod
    - name: my-pod-dir
      persistentVolumeClaim:
        claimName: my-pvc
```

Note that PVCs must be in the same namespace as the related Pod, whereas PVs are not namespaced and are available for all nodes.

### Summary on the Levels of Abstraction
- The Pod requests the volume through a PVC.
- The PVC tries to find a PV in the cluster satisfying the PVCs requirements.
- The PV has the actual storage backend.

The Volume is mounted into the Pod and then from the Pod into the container(s). A Pod (and a container) can mount multiple volumes of different type.

ConfigMap and Secret are local volume types directly managed by K8s (see next video).

### Storage Class
In a cluster with hundreds of Pods needing persistent volume the system administrator would have to define many PVs manually. A tedious task that can quickly get very time consuming and messy. Storage classes provision Persistent Volumes dynamically and automatically whenever a PVC claims it. They are also configured in a yaml file. The storage backend is defined in the `provisioner` attribute:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: slow
provisioner: kubernetes.io/aws-ebs # internal provisioner (prefix kubernetes.io)
parameters:
  type: io1
  iopsPerGB: "10"
  fsType: ext4
```

Another example of a storage class using an external provisioner:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: example-nfs
provisioner: example.com/external-nfs # external provisioner
parameters:
  server: nfs-server.example.com
  path: /share
  readOnly: "false"
```

Storage Class is another abstraction level. It abstracts the underlying storage provider and defines the parameters for that storage.

If you want to let a storage class automatically provision a PV for your PVC, you can add a `storageClassName` attribute to your PVCs `spec`, referencing the required Storage Class. When the Pod then claims storage via the PVC, the PVC requests storage from the Storage Class, which then creates a PV that meets the need of the claim using the provisioner from the actual storage backend.

</details>

*****

<details>
<summary>Video: 12 - ConfigMap & Secret Volume Types</summary>
<br />

With ConfigMap and Secret you can pass individual key-value pairs to your Pods as we've seen in video 7:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service
---
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
type: Opaque
data:
  mongo-root-username: bW9uZ28=
  mongo-root-password: c2VjcmV0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: username
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: password
        - name: ME_CONFIG_MONGODB_SERVER 
          valueFrom: 
            configMapKeyRef:
              name: mongodb-configmap
              key: db_host
```

But how do you pass whole configuration files or certificate files to your Pods?

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mosquitto-config-file
data:
  mosquitto.conf: |
    log_dest stdout
    log_type all
    log_timestamp true
    listener 9001 
---
apiVersion: v1
kind: Secret
metadata:
  name: mosquitto-secret-file
type: Opaque
data:
  secret.file: |
    c29tZSBzdXBlcnNlY3JldCBmaWxlIGNvbnRlbnRzIG5vYm9keSBzaG91bGQgc2Vl
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto
  labels:
    app: mosquitto
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mosquitto
  template:
    metadata:
      labels:
        app: mosquitto
    spec:
      containers:
        - name: mosquitto
          image: eclipse-mosquitto:1.6.2
          ports:
            - containerPort: 1883
          volumeMounts:
            - name: mosquitto-conf
              mountPath: /mosquitto/config
            - name: mosquitto-secret
              mountPath: /mosquitto/secret
              readOnly: true
      volumes:
        - name: mosquitto-conf
          configMap:
            name: mosquitto-config-file
        - name: mosquitto-secret
          secret:
            secretName: mosquitto-secret-file
```

See demo project 2 for more information.

</details>

*****

<details>
<summary>Video: 13 - StatefulSet - Deploying Stateful Applications</summary>
<br />

Stateless applications are deployed in Kubernetes using Deployment, an abstraction over a number of replicated, interchangeable (i.e. from a client's point of view identical) Pods.

Stateful applications are deployed using StatefulSet, which is also an abstraction over a number of replicated Pods, but the Pods are not interchangeable and when a Pod dies and needs to be re-scheduled, it gets the same ID as before. So replication is a more complex task with StatefulSets than it is with Deployments.

The sticky identifier is made up of the name of the StatefuleSet followed by an ordinal number (${statefull-set-name}-${ordinal}).

StatefulSets are used to deploy applications like databases. The first Pod takes the role of the master. Updates are only possible against the master Pod. Every additional Pod that gets scheduled is a worker Pod and synchronizes its data with the previous Pod. Worker Pods can only serve data queries. When Pods need to be down-scheduled, the first one to be deleted is the last created one. That's why fixed identifiers holding an ordinal are needed.

When a Pod of a StatefulSet gets deleted and re-scheduled, it is re-assigned the same volume. So each Pod needs its own PersistentVolume which must use a remote storage system, otherwise it could not be guaranteed, that it is accessable from the new Pod (that might be scheduled on a different Node than its predecessor).

With StatefulSets each Pod gets its individual DNS name (made up of ${pod-name}.${governing-service-name}).

It is important to mention, that a lot of the additional complexity (like configuring the cloning and data synchronization, make remote storage available, make backups, etc.) has to be managed by the developer. Stateful applications and containerized environments are not a really good match.

</details>

*****

<details>
<summary>Video: 14 - Managed Kubernetes Services</summary>
<br />

There are two options to create a Kubernetes cluster on a cloud platform:
- create your own cluster from scratch
- use a managed K8s service

Creating your own cluster is very time consuming as you have to setup and manage all the components (like control plane nodes) and resources (e.g. storage) by yourself.

When you use a managed Kubernetes service of the cloud provider, you only care about (and pay for) the worker nodes. You can use cloud storage solutions and a cloud native load balancer. The disadvantage is, that you depend on cloud provider specific components (vendor lock). This risk can be mitigated using Infrastructure As Code tools like Terraform or Ansible, which abstract away the underlying cloud provider.

### Examples of Managed Kubernetes Services
- AWS: Elastic Kubernetes Service (EKS)
- Azure: Azure Kubernetes Service (AKS) 
- Google: Google Kubernetes Engine (GKE) 
- Linode: Linode Kubernetes Engine (LKE)

</details>

*****

<details>
<summary>Video: 15 - Helm - Package Manager for Kubernetes</summary>
<br />

### Helm as a Package Manager
Helm is the package manager for Kubernetes. Think of it like apt/yum for Kubernetes. It helps in packaging YAML files and distributing them in public and private repositories.

To deploy your application into a K8s cluster you usually have to write a lot of K8s configuration/manifest files. These files can be bundeled into a Helm package which is called Helm Chart. A Helm chart contains a chart description (Chart.yaml) and one or more templates containing K8s manifest files. Helm is the tool managing these charts. Helm charts can be pushed to a Helm repository and downloaded by others needing to deploy ypur application. There a public repos, but you can also have your own private repo in your company (e.g. Nexus).

There are also a lot of official charts available for applications / components of general interest like Mysql, MongoDB, ElasticSearch or Prometheus. So if you want to use these components in your K8s cluster, you can just download the related Helm charts (often provided by the official creator of the component) and don't have to write all the needed K8s configuration files by yourself.

To find existing charts you can either execute `helm search <keywords>` on the command line or go to [Helm Hub](https://artifacthub.io/) and browse available packages.

### Helm as a Templating Engine
When deploying a couple of microservices, you often write K8s configurations files that are more or less the same and differ only in a few lines. With Helm Charts you can write template files containing the common part and placeholders like `{{ .Values.container.image }}`, from which the concrete configuration files will then be generated. The values are defined in a separate file called `values.yaml`.\
Another use case is to deploy the same bundle of K8s components across multiple environments (e.g. Development, Staging, Production).

### Helm Chart Structure
A Helm Chart directory has the following structure:
```txt
mychart/          -> top level folder defining the name of the chart
  Chart.yaml      -> meta info about the chart
  values.yaml     -> values needed in the template files
  charts/         -> dependencies (oder charts)
  templates/      -> the actual template files of this chart
  ...
```

To create the K8s configuration files execute `helm install --values=overlays/dev/values.yaml <chartname>`. The `values.yaml` file contains the default values. They are overridden/merged with the values in a file provided with the `--values` option resulting in the final `.Values` object referenced in the template files. As an alternative (for quick changes) it is also possible to override values with the `--set` option like this: `helm install --set version=2.0.0 <chartname>`.

### Helm Release Management
**Helm Version 2**\
Helm is divided into two parts, a Helm Client (CLI) and Server (called Tiller). Tiller is running in the K8s cluster where you want to deploy your components. When `helm install <chartname>` is executed, the YAML files are sent to the Tiller, which applies them on the K8s cluster. Tiller stores all the files it received thus creating a history of chart executions. You may then call `helm upgrade <chartname>` to just apply the changes to the existing deployment instead of creating a new one. You may also rollback to earlier versions using the `helm rollback <chartname>` command.

**Helm Version 3**\
Because Tiller has too much power inside the K8s cluster (it may create, update, delete components), it was seen as too big a security issue and got removed completely in Helm version 3.

In Helm 3, an application's state is tracked in-cluster by a pair of objects:
- the release object: represents an instance of an application
- the release version secret: represents an application's desired state at a particular instance of time (the release of a new version, for example)

A `helm upgrade` requires an existing release object (which it may modify) and creates a new release version secret that contains the new values and rendered manifest.

### Links
- [Install Helm](https://helm.sh/docs/intro/install/)
- [Helm Hub](https://artifacthub.io/)

</details>

*****

<details>
<summary>Video: 16 - Helm Demo - Managed K8s cluster</summary>
<br />

### Create a K8s Cluster on Linode Kubernetes Engine (LKE)
Login to your [Linode account](https://cloud.linode.com/), press the blue "Create" button and select "Kubernetes". Enter a cluster name (e.g. 'devops-bootcamp'), choose a region close to you (e.g. 'Frankfurt, DE (eu-central)') and select the latest Kubernetes version (e.g. 1.26). In the "Add Node Pools" section select the "Shared CPU" tab and add 2 "Linode 4 GB" nodes to the cart. Check the "I have read..." disclaimer and press the "Create Cluster" button.\
On the dashboard you can see the two worker nodes (Linodes). Wait until both are up and running.

In the Kubernetes section at the top you can download a 'devops-bootcamp-kubeconfig.yaml' file with the credentials and certificates you need to connect to the K8s cluster. Download it and set the environment variable KUBECONFIG on your local machine to this file:
```sh
export KUBECONFIG=</path/to/download-folder>/devops-bootcamp-kubeconfig.yaml

# now kubectl commands will be connected with the linode cluster
kubectl get nodes
# =>
# NAME                            STATUS   ROLES    AGE   VERSION
# lke104424-156177-6445973ec1e1   Ready    <none>   19m   v1.26.3
# lke104424-156177-6445973f23f0   Ready    <none>   19m   v1.26.3
```

### Deploy a Replicated Stateful MongoDB Service in the Cluster Using a Helm Chart
To deploy a StatefulSet for the MongoDB service we could create all the needed K8s configuration files by ourselves. Or we could use a bundle (Helm Chart) of those files.

If you haven't installed Helm yet, [install it now](https://helm.sh/docs/intro/install/). On a Mac, the easiest way to install Helm is to execute
```sh
brew update
brew install helm
```

Google for MongoDB Helm Charts. You should find the charts maintained by [Bitnami](https://bitnami.com/stack/mongodb/helm). Execute the following commands:
```sh
# add the bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# search for mongodb charts in this repo
helm search repo bitnami/mongodb
# =>
# NAME                   	CHART VERSION	APP VERSION	DESCRIPTION                                       
# bitnami/mongodb        	13.9.4       	6.0.5      	MongoDB(R) is a relational open source NoSQL da...
# bitnami/mongodb-sharded	6.3.3        	6.0.5      	MongoDB(R) is an open source NoSQL database tha...
```

To see the parameters of the chart, open the browser and navigate to `https://github.com/bitnami/charts/tree/main/bitnami/mongodb`. You'll find that there are parameters `architecture`, `replicaCount`, `persistence.storageClass` and `auth.rootPassword` (among many others). To override these parameters create a file called `devops-bootcamp-mongodb-values.yaml` with the following content:
```yaml
architecture: replicaset
replicaCount: 3
persistence:
    storageClass: "linode-block-storage"
auth:
    rootPassword: t0p-secret
```

To install the chart in our Linode cluster execute the following commands:
```sh
helm install mongodb --values devops-bootcamp-mongodb-values.yaml bitnami/mongodb
# =>
# NAME: mongodb
# LAST DEPLOYED: Sun Apr 23 23:47:38 2023
# NAMESPACE: default
# STATUS: deployed
# REVISION: 1
# TEST SUITE: None
# NOTES:
# CHART NAME: mongodb
# CHART VERSION: 13.9.4
# APP VERSION: 6.0.5
# 
# ** Please be patient while the chart is being deployed **
# 
# MongoDB&reg; can be accessed on the following DNS name(s) and ports from within your cluster:
# 
#     mongodb.default.svc.cluster.local
# 
# To get the root password run:
#
#    export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace default mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 -d)
#
# To connect to your database, create a MongoDB&reg; client container:
#
#    kubectl run --namespace default mongodb-client --rm --tty -i --restart='Never' --env="MONGODB_ROOT_PASSWORD=$MONGODB_ROOT_PASSWORD" --image docker.io/bitnami/mongodb:6.0.5-debian-11-r4 --command -- bash
#
# Then, run the following command:
#    mongosh admin --host "mongodb" --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD
#
# To connect to your database from outside the cluster execute the following commands:
#
#    kubectl port-forward --namespace default svc/mongodb 27017:27017 &
#    mongosh --host 127.0.0.1 --authenticationDatabase admin -p $MONGODB_ROOT_PASSWORD

kubectl get statefulset --watch
```

In the Linode web console you should see that three persistent volumes have been created (one for each replica) and attached to the two worker nodes.

### Deploy MongoExpress
Create a file called `devops-bootcamp-mongo-express.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports: 
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_SERVER
          value: mongodb-0.mongodb-headless
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          value: root # kubectl get pods; kubectl exec -it mongodb-0 -- env | grep MONGODB
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD
          valueFrom: 
            secretKeyRef:
              name: mongodb              # kubectl get secrets
              key: mongodb-root-password # kubectl get secret mongodb -o yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
```

To find out some of the configuration values you can inspect the current state of the K8s cluster. To get the MongoDB admin username for example, execute the following commands:
```sh
kubectl get pods
# => 
# NAME                READY   STATUS    RESTARTS   AGE
# mongodb-0           1/1     Running   0          20h
# mongodb-1           1/1     Running   0          20h
# mongodb-2           1/1     Running   0          20h
# mongodb-arbiter-0   1/1     Running   0          20h

kubectl exec -it mongodb-0 -- env | grep MONGODB
# => 
# MONGODB_REPLICA_SET_NAME=rs0
# MONGODB_DISABLE_JAVASCRIPT=no
# MONGODB_REPLICA_SET_KEY=mAiQWTl6Xa
# MONGODB_ENABLE_JOURNAL=yes
# MONGODB_INITIAL_PRIMARY_HOST=mongodb-0.mongodb-headless.default.svc.cluster.local
# MONGODB_ROOT_PASSWORD=t0p-secret
# MONGODB_DISABLE_SYSTEM_LOG=no
# MONGODB_PORT_NUMBER=27017
# MONGODB_SYSTEM_LOG_VERBOSITY=0
# MONGODB_ADVERTISED_HOSTNAME=mongodb-0.mongodb-headless.default.svc.cluster.local
# MONGODB_ROOT_USER=root   <---------
# MONGODB_ENABLE_IPV6=no
# MONGODB_ENABLE_DIRECTORY_PER_DB=no
```

Or to get the secret key for the root password:
```sh
kubectl get secrets
# => 
# NAME                            TYPE                                  DATA   AGE
# default-token-2l7j9             kubernetes.io/service-account-token   3      23h
# mongodb                         Opaque                                2      21h
# mongodb-token-ds6qr             kubernetes.io/service-account-token   3      21h
# sh.helm.release.v1.mongodb.v1   helm.sh/release.v1                    1      21h

kubectl get secret mongodb -o jsonpath="{.data}"
# => {"mongodb-replica-set-key":"bUFpUVdUbDZYYQ==","mongodb-root-password":"dDBwLXNlY3JldA=="}
```

Apply the file, wait until the pod is running and make sure the server is listening for incoming requests:
```sh
kubectl apply -f devops-bootcamp-mongo-express.yaml

kubectl get pods
# =>
# NAME                             READY   STATUS    RESTARTS   AGE
# mongo-express-78d8b477c4-xtncr   1/1     Running   0          17s
# mongodb-0                        1/1     Running   0          21h
# mongodb-1                        1/1     Running   0          21h
# mongodb-2                        1/1     Running   0          21h
# mongodb-arbiter-0                1/1     Running   0          21h

kubectl logs mongo-express-78d8b477c4-xtncr
# =>
# ...
# Mongo Express server listening at http://0.0.0.0:8081
```

### Deploy Ingress Controller
You can install the K8s managed nginx-ingress controller using Helm:
```sh
helm repo add ingress https://kubernetes.github.io/ingress-nginx
helm install ingress-nginx ingress/ingress-nginx
# wait until IP address has been assigned
kubectl get service ingress-nginx-controller --watch
# => 
# NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
# ingress-nginx-controller   LoadBalancer   10.128.84.113   143.42.222.35   80:31566/TCP,443:30678/TCP   6m35s
```

When you go to the Linode management web-console and open the NodeBalancers section, you should see that there was one NodeBalancer created and its IP address is 143.42.222.35, the EXTERNAL-IP of the ingress LoadBalancer service.

### Create an Ingress Rule for Accessing MongoExpress from the Browser
Create a file called `devops-bootcamp-ingress.yaml` with the following content:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: mongo-express
spec:
  rules:
    - host: 143-42-222-35.ip.linodeusercontent.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: mongo-express-service
                port: 
                  number: 8081
```

Apply the file to the cluster:
```sh
kubectl apply -f devops-bootcamp-ingress.yaml

# check it
kubectl get ingress
# =>
# NAME            CLASS    HOSTS                                    ADDRESS         PORTS   AGE
# mongo-express   <none>   143-42-222-35.ip.linodeusercontent.com   143.42.222.35   80      59s
```

Now you can access the MongoExpress application in your browser navigating to [http://143-42-222-35.ip.linodeusercontent.com](http://143-42-222-35.ip.linodeusercontent.com). Create a new database and collection in it.

Then delete the Pods by scaling down the replicas to zero:
```sh
kubectl scale --replicas=0 statefulset/mongodb

kubectl get pods
# =>
# NAME                                        READY   STATUS        RESTARTS   AGE
# ingress-nginx-controller-5c6fd54c59-cwnwl   1/1     Running       0          33m
# mongo-express-78d8b477c4-xtncr              1/1     Running       0          60m
# mongodb-0                                   1/1     Running       0          22h
# mongodb-1                                   1/1     Running       0          22h
# mongodb-2                                   1/1     Terminating   0          22h
# mongodb-arbiter-0                           1/1     Running       0          22h
# ...
# NAME                                        READY   STATUS    RESTARTS   AGE
# ingress-nginx-controller-5c6fd54c59-cwnwl   1/1     Running   0          34m
# mongo-express-78d8b477c4-xtncr              1/1     Running   0          60m
# mongodb-arbiter-0                           1/1     Running   0          22h
``` 

In the Linode management web-console you will find that all the volumes have been unattached but are still there.

Now re-create the Pods by scaling the replicas up to 3 again
```sh
kubectl scale --replicas=3 statefulset/mongodb
```
and watch the volumes being re-attached to the new Pods in the Linode management web-console.

If you want to remove all the components created via Helm Charts, just execute:
```sh
helm uninstall ingress-nginx
helm uninstall mongodb
```

Note that the PersistentVolumes that have been created when installing the mongodb chart, don't get deleted by `helm uninstall`. This is a security feature, to keep the data in case you want to re-install the database later. If you want to delete the volumes, you'll have to do it manually in the Linode management web-console.

When you are done with the Kubernetes cluster on Linode, just delete it. Note that the volumes won't be deleted even if you delete the whole K8s cluster. You'll have to delete them manually.

</details>

*****

<details>
<summary>Video: 17 - Deploying Images in Kubernetes from private Docker repository</summary>
<br />

To pull a Docker image from a private repository into your K8s cluster, you have to configure explicit access from the K8s cluster to the private Docker registry. So you have to create a Secret component in the K8s cluster containing the credentials for the Docker registry. And then you have to use this Secret in the Deployment/Pod (`imagePullSecrets`).

For the demo there is a Docker image available in a private AWS ECR registry. And K8s is running in a local minikube cluster.

### Create a Secret component holding the Credentials for the AWS ECR registry
To create a Secret holding the credentials for docker-login, we create a configuration file of the following type:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: <secret-name>
data:
  .dockerconfigjson: <base64-encoded-contents-of ~/.docker/config.json file>
type: kubernetes.io/dockerconfigjson
```

To get the `~/.docker/config.json` file for our private AWS ECR registry, we have to login to that registry. We can either get the password and the do a docker login like this: 
```sh
aws ecr get-login-password --region <your region>
# => pwd
docker login -u AWS -p <pwd> <registry-host>
```

or we can directly execute the following command:
```sh
aws ecr get-login-password --region <your region> | docker login --username AWS --password-stdin <registry-host>
```

You can find this command in the AWS management web console. Navigate to Amazon ECR > Repositories, click on your repository and press the "View push commands" button.

The second way is the preferred one because you don't have to type in your ECR password in your command line (writing it to the command line history).

Executing the above docker-login command created an entry in the file `~/.docker/config.json` that looks like this:
```json
{
    "auths": {
        "<registry-host>": {},
    },
    "credsStore": "osxkeychain"
}
```

If we put this content base64 encoded into the Secret component, we won't be able to use it in our minikube cluster, because the minikube cluster is running in its own docker container and does not have access to the osx keychain. So we have to execute the docker-login command from within the minikube container.

```sh
minikube ssh

pwd
# => /home/docker

docker login -u AWS -p <pwd> <registry-host>
cat .docker/config.json
# =>
# {
# 	"auths": {
# 		"<registry-host>": {
# 			"auth": "QVdTOmV5SndZWGxzYjJGa0lqb2.....ESTFPRFEyT1RWOQ=="
# 		}
# 	}
# }

exit
```

Now we have to copy that config.json from the minikube container to our ~/.docker folder:
```sh
scp -i $(minikube ssh-key) docker@$(minikube ip):.docker/config.json ~/.docker/minikube-config.json
```

****
Note: This command does not work with minikube running in a docker container. You won't be able to connect to the IP address returned by `minikube ip`. To get the content we have to either copy it manually or mount a directory into minikube:
```sh
minikube mount ~/.docker:/home/docker/.docker

minikube ssh
docker login -u AWS -p eyJwYXlsb2FkI.....ODI2NjgyMDF9 369076538622.dkr.ecr.eu-central-1.amazonaws.com
exit

cat ~/.docker/config.json
```
****

With these preparations we are ready to get the base64 encoded content of the file `~/.docker/minikube-config.json` and create a Secret from it:
```sh
cat ~/.docker/minikube-config.json | base64
# => ewoJImF1dGhzIjogewoJCSI.....T09IgoJCX0KCX0KfQ==
```

Create a file called `docker-secret.yaml` with the following content:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-registry-key
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSI.....T09IgoJCX0KCX0KfQ==
type: kubernetes.io/dockerconfigjson
```

Apply it with `kubectl apply -f docker-secret.yaml`.

Another way of creating the Secret, once you have the config.json file, is to use the following kubectl command:
```sh
kubectl create secret generic my-registry-key \
  --from-file=.dockerconfigjson=.docker/config.json \
  --type=kubernetes.io/dockerconfigjson
```

And finally you can also combine the docker login command with the create secret command:
```sh
kubectl create secret docker-registry my-registry-key \
  --docker-server=<registry-host> \
  --docker-username=AWS \
  --docker-password=<pwd>
```

This command does a login to the private docker registry using the given username and password and creates a Secret containing the base64 encoded content of the config.json file that would have been created by a regular `docker login` command. Even if this last way of creating the secret is the most comfortable one, be aware that you cannot create a secret containing the credentials for multiple registries. With the previous two ways you can login to multiple docker registries resulting in a config.json file that contains the credentials for all of them. 

### Create a Deployment Component Using the Docker Registry Secret
Create a file called `my-app-deployment.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      imagePullSecrets:
      - name: my-registry-key
      containers:
      - name: my-app
        image: <registry-host>/my-app:1.0
        imagePullPolicy: Always
        ports:
          - containerPort: 3000
```

Apply it to the cluster:
```sh
kubectl apply -f my-app-deployment.yaml

kubectl get pods
kubectl describe pod <pod-name>
# => Events show you whether the image was pulled successfully or not
```

Final note: The secret have to be in the same namespace as the deployment using it. So if you have three deployments in three different namespaces and all of them pull an image from the same private repository, you have to create three different secrets (containing the same .docker/config.json file) in all three namespaces.

</details>

*****

<details>
<summary>Video: 18 - Kubernetes Operators for Managing Complex Applications</summary>
<br />

Stateless applications are totally managed by K8s. There is no application specific knowledge needed for controlling the lifecycle of these components. The control loop contantly checks the actual state, compares it with the desired state and tries to resolve differences.

With stateful applications, K8s can't automate this process natively. Stateful applications need constant management and syncing after deployment. So stateful applications, like databases, need to be operated. But instead of a human operator, K8s operators are automated and scripted. They can be seen as a custom control loop, that watches for changes in the application state. Did a replica die? It creates a new one. Did an application configuration change? It applies the up-to-date configuration. Did the application image version get updated? It restarts it with a new image version. For all that it makes use of CRDs (Custom Resource Definition), which are custom K8s components. It includes domain or application-specific knowledge, e.g. for a MySql database this could be:
- how to create mysql cluster
- how to run it
- how to synchronize the data
- how to update
- how to do backups
- etc.

Operators are often created / provided by the official maintainers of an application (like prometheus, postgres, elastic-search, mysql, etcd store, ...). There is an [OperatorHub](https://operatorhub.io/) website, or you can find operators on GitHub, etc... If you want / need to write your own operator, there's also an Operator SDK available, that supports you in this task.

</details>

*****

<details>
<summary>Video: 19 - Helm and Operator Demo</summary>
<br />

### Prometheus Architecture
- The **Prometheus Server** retrieves, processes and stores metrics data and provides HTTP endpoints accepting PromQL queries.
- The **Alertmanager** can be used to send alerts (e-Mail, Slack, etc.) based on the data the Prometheus server collected.

### How to deploy the various Prometheus components in a K8s cluster?
You can either manually create all the configuration files for the needed components (but there are a lot of components, so this is a very time consuming way of deploying Prometheus) and apply them in the right order, or you can use an operator that manages the combination of all the compontents that make up the Prometheus stack as one unit. The most effitient way of deploying an operator is using a Helm chart solving this task. The Helm chart will do the initial setup of the Prometheus Operator, Prometheus Server and Alertmanager, and the operator will then manage the running Prometheus setup.

### Setup Prometheus with a Helm Chart
#### Prometheus Community
Open your browser and navigate to the [Prometheus Community on Github](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack). Here you'll find the commands to install the Helm Chart:

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
# =>
# kube-prometheus-stack has been installed. Check its status by running:
kubectl --namespace default get pods -l "release=prometheus"

# Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

****
Alternative: **Bitnami**
Open your browser and navigate to the [Bitnami Prometheus Operator](https://bitnami.com/stack/prometheus-operator/helm) website. Here you'll find the commands to install the Helm Chart:

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install prometheus bitnami/kube-prometheus
# =>
# Watch the Prometheus Operator Deployment status using the command:
kubectl get deploy -w --namespace default -l app.kubernetes.io/name=kube-prometheus-operator,app.kubernetes.io/instance=prometheus

# Watch the Prometheus StatefulSet status using the command:
kubectl get sts -w --namespace default -l app.kubernetes.io/name=kube-prometheus-prometheus,app.kubernetes.io/instance=prometheus

# Prometheus can be accessed via port "9090" on the following DNS name from within your cluster:
# prometheus-kube-prometheus-prometheus.default.svc.cluster.local

# To access Prometheus from outside the cluster execute the following commands:
echo "Prometheus URL: http://127.0.0.1:9090/"
kubectl port-forward --namespace default svc/prometheus-kube-prometheus-prometheus 9090:9090

# Watch the Alertmanager StatefulSet status using the command:
kubectl get sts -w --namespace default -l app.kubernetes.io/name=kube-prometheus-alertmanager,app.kubernetes.io/instance=prometheus

# Alertmanager can be accessed via port "9093" on the following DNS name from within your cluster:
# prometheus-kube-prometheus-alertmanager.default.svc.cluster.local

# To access Alertmanager from outside the cluster execute the following commands:
echo "Alertmanager URL: http://127.0.0.1:9093/"
kubectl port-forward --namespace default svc/prometheus-kube-prometheus-alertmanager 9093:9093
``` 

For detailed information on all the parameters of the Helm chart, open the [Bitnami Prometheus Helm Charts on Github](https://github.com/bitnami/charts/tree/main/bitnami/kube-prometheus/) page.
****

### Understand the created Prometheus components
```sh
kubectl get all
```
There are
- 2 Stateful Sets: The Prometheus Server and the Alertmanager
- 3 Deployments: The Prometheus Operator (that created the two Stateful Sets), Grafana and Kube State Metrics (a dependency of the Prometheus Operator Helm Chart scraping K8s components)
- 3 ReplicaSets created by the deployments
- 1 Node Exporter Daemon Set (Daemon Sets run on every worker node) connecting to every worker node and translating node metrics (like CPU usage, load on the server, etc...) to Prometheus metrics
- a bunch of Pods
- a bunch of Services
- a bunch of ConfigMaps (e.g. how to connect to default metrics, prometheus rulefiles)
- a bunch of Secrets (certificates, usernames & passwords for Grafana, Prometheus, Operator, etc.)
- a bunch of CRDs (Custom Resource Definitions)

So besides the whole monitoring stack being deployed, we also get a monitoring configuration for the K8s cluster including both nodes and components monitoring.

You don't have to know all the details of these components. Important to know is how to add/adjust alert rules and how to adjust Prometheus configuration.

```sh
kubectl get statefulsets
# =>
# NAME                                                   READY   AGE
# alertmanager-prometheus-kube-prometheus-alertmanager   1/1     22h
# prometheus-prometheus-kube-prometheus-prometheus       1/1     22h

kubectl describe statefulset prometheus-prometheus-kube-prometheus-prometheus > prometheus.yaml
# =>
# ...
# Volumes:
#    config:
#     Type:        Secret (a volume populated by a Secret)
#     SecretName:  prometheus-prometheus-kube-prometheus-prometheus
#     Optional:    false
# ...
#    prometheus-prometheus-kube-prometheus-prometheus-rulefiles-0:
#     Type:      ConfigMap (a volume populated by a ConfigMap)
#     Name:      prometheus-prometheus-kube-prometheus-prometheus-rulefiles-0
#     Optional:  false

kubectl get secret prometheus-prometheus-kube-prometheus-prometheus -o jsonpath="{.data.prometheus\.yaml\.gz}" | base64 -d | gunzip > prometheus-config.yaml

kubectl get configmap prometheus-prometheus-kube-prometheus-prometheus-rulefiles-0 -o yaml > rules.yaml
```

### Access Grafana
```sh
kubectl get services
# => 
# NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
# ...
# prometheus-grafana   ClusterIP   10.107.198.230   <none>        80/TCP    22h
# ...
```

Grafana is an internal service. In production environment you would usually define an ingress to make it accessible from outside of the cluster. To access it for this demo, we just define a port-forwarding:
```sh
kubectl get pods
# =>
# NAME                                                     READY   STATUS    RESTARTS        AGE
# alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   1 (4m50s ago)   4m53s
# prometheus-grafana-6984c5759f-cd6zw                      3/3     Running   0               4m56s
# prometheus-kube-prometheus-operator-655c5b45c7-tlfvc     1/1     Running   0               4m56s
# prometheus-kube-state-metrics-7fbdd95dc4-bllf5           1/1     Running   0               4m56s
# prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0               4m53s
# prometheus-prometheus-node-exporter-jw2gg                1/1     Running   0               4m56s

kubectl port-forward prometheus-grafana-6984c5759f-cd6zw 3000
```

Now we can access the Grafana UI in the browser on `localhost:3000`, username `admin` and password `prom-operator` (looked up in the prometheus operator [chart documentation](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml) -> `adminPassword`).

### Access Prometheus UI
Prometheus itself provides a UI too. It can be access on port 9090 of the related pod, so let's setup a port-forwarding too:
```sh
kubectl port-forward prometheus-prometheus-kube-prometheus-prometheus-0 9090
```

Now access the Prometheus UI on `localhost:9090`. Under 'Status' > 'Rules' you can find all the alerting rules and under 'Status' > 'Targets' you can find all the metric endpoints that are being scraped.

</details>

*****

<details>
<summary>Video: 20 - Secure your cluster - Authorization with RBAC</summary>
<br />

### Authentication
Cluster administrators can choose from different authentication strategies like
  - static token file (a csv file with a minimum of 3 columns: token, user name, user uid, followed by optional group names)
  - client certificates
  - 3rd party identity service (like LDAP)

The API server (part of the K8s control plane) handles authentication of all the requests using the configured authentcation strategy.

### Authorization
K8s supports multiple authorization modules, such as
- ABAC mode (Attribute Based Access Control)
- RBAC mode (Role Based Access Control)
- Webhook mode

On cluster creation, admins configure the authorization modules that should be used. K8s checks each module, and if any module authorizes the request, then the request can proceed.

### Role & RoleBinding
The **Role** component allows you to define namespace permissions. It is bound to a specific namespace and allows you to specify what resources you can in that namespace and what actions can be performed on these resources (access permissions).

Linking a role to a specific person / user or a group of users is the purpose of the **RoleBinding** component.

### ClusterRole & ClusterRoleBinding
As Role is bound to a namespace it cannot be used for administrators since they need cluster wide permissions. **ClusterRole** and **ClusterRoleBinding** are similar to Role and RoleBinding but not related to a namespace but to a whole cluster and can thus be used to define the permissions for admins and admin groups.

### Users, Groups and Service-Accounts in Kubernetes
K8s doesn't manage users natively. There is no K8s resource representing normal user accounts.

Applications and services also access the cluster. For this purpose there is (unlike for human users) a specific K8s component representing applications called **ServiceAccount**. It can be created like this:
`kubectl create serviceaccount <service-account-name>`. ServiceAccounts can be bound to Roles or ClusterRoles via RoleBinding or ClusterRoleBinding.

### Example Configuration Files
See the [K8s Documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

**Role**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "create", "list"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["my-configmap"] # restrict permissions to a specific ConfigMap only
  verbs: ["update", "get"]
```

**RoleBinding**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
- kind: ServiceAccount
  name: jenkins
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role # this must be Role or ClusterRole
  name: developer # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

Note that a RoleBinding can also reference a ClusterRole to grant the permissions defined in that ClusterRole to resources inside the RoleBinding's namespace. This kind of reference lets you define a set of common roles across your cluster, then reuse them within multiple namespaces.

**ClusterRole**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: cluster-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "create", "list", "delete", "update"]
```

**ClusterRoleBinding**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
subjects:
- kind: Group
  name: cluster-admins
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

### Checking API Access
Kubectl provides an `auth can-i` subcommand to quickly check wether the current user can perform a given action:
```sh
kubectl auth can-i create deployments --namespace dev
```

Admins can also check the permissions of other users:
```sh
kubectl auth can-i list secrets --namespace dev --as dave
kubectl auth can-i list pods --namespace target --as system:serviceaccount:dev:dev-sa
```

### Links
- [Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/)
- [3 Realistic Approaches to K8s RBAC](https://thenewstack.io/three-realistic-approaches-to-kubernetes-rbac/)

</details>

*****

<details>
<summary>Video: 21 - Microservices in Kubernetes</summary>
<br />

### Introduction to Microservices
- Microservices are an architectural approach to software development.
- Software is composed of small independent services (instead of having a huge monolith).
- Each business functionality is encapsulated into own Microservice (MS).

Benefits:
- Each MS can be developed, packaged and released independently.
- Changes in one MS don't affect other MS.
- Less interconnected logic, loosely coupled
- Each MS can be developed by separate developer teams.

### Communication between Microservices
The communication between microservices can happen in different ways:
- Service-to-service API calls (direct communication)
- Message-based communication (through a message broker, like Rabbitmq or Redis)
- Service-mesh architecture (each MS has its own helper proxy / sidecar container handling the communication for that specific MS, e.g. Istio, Linkerd, HashiCorp Consul)

### Deploying a Microserices Application
To successfully deploy a MS application into  a K8s cluster, you need to know
- which services you need to deploy
- which service is talking to which other service
- how are they communicating
- which database (or other 3rd party services) are they using
- on which port does each service run

Then you have to prepare the K8s environment by deploying any needed 3rd party apps and creating the Secrets and ConfigMaps for the microservices. And finally you can create Deployment and Service for each microservice.

When deploying multiple similar services to K8s, you can use a helm chart with one common template and replace specific values for each service on the fly during deployment.

</details>

*****

<details>
<summary>Video: 22 - Demo project: Deploy Microservices Application</summary>
<br />

We are going to deploy the microservices application in the GitHub repository [nanuchi/microservices-demo](https://github.com/nanuchi/microservices-demo) into a Linode K8s cluster. To do this, we don't need to know and understand the source code of the various microservices. What you need to know is
- what microservices you have to deploy (image names)
- what environment variables they need
- on which ports they are listening
- how they are connected
- what 3rd party services or databases they are using
- which service(s) must be accessible from outside of the cluster

Developers tell us that the application is made up of the following microservices:
1. Frontend: entrypoint, accessible from outside; port 8080; talking to 2, 3, 4, 7, 9, 10
2. AdService: port 9555
3. CheckoutService: port 5050; talking to 4, 5, 6, 7, 8, 10
4. CurrencyService: port 7000
5. EmailService: port 8080
6. PaymentService: port 50051
7. ShippingService: port 50051
8. ProductCatalogService: port 3550
9. RecommendationService: port 8080; talking to 8
10. CartService: port 7070; storing data in Redis In-Memory cache
11. LoadGenerator: not needed in production, just needed for doing load tests, talking to 1

Since all the microservices are developped by one single team we decide to deploy them all into the same namespace.

### Create Deployment and Service Configurations
```sh
mkdir online-shop-microservices
cd online-shop-microservices
touch config.yaml
```

We start with specifying the Deployment and Service for the email microservice. Add the following content to the `config.yaml` file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: emailservice
spec:
  selector:
    matchLabels:
      app: emailservice
  template:
    metadata:
      labels:
        app: emailservice
    spec:
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/emailservice
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
---
apiVersion: v1
kind: Service
metadata:
  name: emailservice
spec:
  type: ClusterIP
  selector:
    app: emailservice
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 8080
```

Since the example microservices application is a fork of a demo application provided by Google, the images are available in a [public Google image registry](https://console.cloud.google.com/gcr/images/google-samples/global/microservices-demo). For a real application developped by you, you would probably deploy your images in a private Docker registry of your company.

Each microservice container needs an environment variable called `PORT` specifying the port the container is listening on.

We continue with the recommendation microservice. Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recommendationservice
spec:
  selector:
    matchLabels:
      app: recommendationservice
  template:
    metadata:
      labels:
        app: recommendationservice
    spec:
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/recommendationservice
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        - name: PRODUCT_CATALOG_SERVICE_ADDR
          value: "productcatalogservice:3550"
---
apiVersion: v1
kind: Service
metadata:
  name: recommendationservice
spec:
  type: ClusterIP
  selector:
    app: recommendationservice
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
```

Since the recommendation microservice is talking to the product-catalog microservice we have to tell it under which hostname and port this service can be accessed. Since all the Services are running in the same namespace, their hostname is just the service name. We are going to name the Service for the product-catalog microservice `productcatalogservice`. It will be listening on port 3550 (attribute `port`). The address of the product-catalog microservice's Service is stored in the environment variable `PRODUCT_CATALOG_SERVICE_ADDR`.

The next microservices to be configured are the payment microservice, the product-catalog, the currency, the shipping and the ad microservices. There's nothing special with these microservices. Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: paymentservice
spec:
  selector:
    matchLabels:
      app: paymentservice
  template:
    metadata:
      labels:
        app: paymentservice
    spec:
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/paymentservice
        ports:
        - containerPort: 50051
        env:
        - name: PORT
          value: "50051"
        - name: DISABLE_PROFILER
          value: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: paymentservice
spec:
  type: ClusterIP
  selector:
    app: paymentservice
  ports:
  - protocol: TCP
    port: 50051
    targetPort: 50051
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productcatalogservice
spec:
  selector:
    matchLabels:
      app: productcatalogservice
  template:
    metadata:
      labels:
        app: productcatalogservice
    spec:
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/productcatalogservice
        ports:
        - containerPort: 3550
        env:
        - name: PORT
          value: "3550"
---
apiVersion: v1
kind: Service
metadata:
  name: productcatalogservice
spec:
  type: ClusterIP
  selector:
    app: productcatalogservice
  ports:
  - protocol: TCP
    port: 3550
    targetPort: 3550
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: currencyservice
spec:
  selector:
    matchLabels:
      app: currencyservice
  template:
    metadata:
      labels:
        app: currencyservice
    spec:
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/currencyservice
        ports:
        - containerPort: 7000
        env:
        - name: PORT
          value: "7000"
        - name: DISABLE_PROFILER
          value: "1"
---
apiVersion: v1
kind: Service
metadata:
  name: currencyservice
spec:
  type: ClusterIP
  selector:
    app: currencyservice
  ports:
  - protocol: TCP
    port: 7000
    targetPort: 7000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shippingservice
spec:
  selector:
    matchLabels:
      app: shippingservice
  template:
    metadata:
      labels:
        app: shippingservice
    spec:
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/shippingservice
        ports:
        - containerPort: 50051
        env:
        - name: PORT
          value: "50051"
---
apiVersion: v1
kind: Service
metadata:
  name: shippingservice
spec:
  type: ClusterIP
  selector:
    app: shippingservice
  ports:
  - protocol: TCP
    port: 50051
    targetPort: 50051
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: adservice
spec:
  selector:
    matchLabels:
      app: adservice
  template:
    metadata:
      labels:
        app: adservice
    spec:
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/adservice
        ports:
        - containerPort: 9555
        env:
        - name: PORT
          value: "9555"
---
apiVersion: v1
kind: Service
metadata:
  name: adservice
spec:
  type: ClusterIP
  selector:
    app: adservice
  ports:
  - protocol: TCP
    port: 9555
    targetPort: 9555
```

The next microservice to be configured is the cart-service. It does not define a `PORT` environment variable but a `REDDIS_ADDR` instead. The Redis cart will be configured at the end. Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cartservice
spec:
  selector:
    matchLabels:
      app: cartservice
  template:
    metadata:
      labels:
        app: cartservice
    spec:
      containers:
      - name: server
        image: gcr.io/google-samples/microservices-demo/cartservice
        ports:
        - containerPort: 7070
        env:
        - name: REDIS_ADDR
          value: "redis-cart:6379"
---
apiVersion: v1
kind: Service
metadata:
  name: cartservice
spec:
  type: ClusterIP
  selector:
    app: cartservice
  ports:
  - protocol: TCP
    port: 7070
    targetPort: 7070
```

And now we configure the Checkout micorservice and the Frontend microservice. Both talk to a lot of other microservices and thus need a lot of environment variables defining the according Service addresses:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkoutservice
spec:
  selector:
    matchLabels:
      app: checkoutservice
  template:
    metadata:
      labels:
        app: checkoutservice
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/checkoutservice
          ports:
          - containerPort: 5050
          env:
          - name: PORT
            value: "5050"
          - name: PRODUCT_CATALOG_SERVICE_ADDR
            value: "productcatalogservice:3550"
          - name: SHIPPING_SERVICE_ADDR
            value: "shippingservice:50051"
          - name: PAYMENT_SERVICE_ADDR
            value: "paymentservice:50051"
          - name: EMAIL_SERVICE_ADDR
            value: "emailservice:5000"
          - name: CURRENCY_SERVICE_ADDR
            value: "currencyservice:7000"
          - name: CART_SERVICE_ADDR
            value: "cartservice:7070"
---
apiVersion: v1
kind: Service
metadata:
  name: checkoutservice
spec:
  type: ClusterIP
  selector:
    app: checkoutservice
  ports:
  - protocol: TCP
    port: 5050
    targetPort: 5050
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: server
          image: gcr.io/google-samples/microservices-demo/frontend
          ports:
          - containerPort: 8080
          env:
          - name: PORT
            value: "8080"
          - name: PRODUCT_CATALOG_SERVICE_ADDR
            value: "productcatalogservice:3550"
          - name: CURRENCY_SERVICE_ADDR
            value: "currencyservice:7000"
          - name: CART_SERVICE_ADDR
            value: "cartservice:7070"
          - name: RECOMMENDATION_SERVICE_ADDR
            value: "recommendationservice:8080"
          - name: SHIPPING_SERVICE_ADDR
            value: "shippingservice:50051"
          - name: CHECKOUT_SERVICE_ADDR
            value: "checkoutservice:5050"
          - name: AD_SERVICE_ADDR
            value: "adservice:9555"
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - name: http
    port: 80
    targetPort: 8080
    nodePort: 30007
```

The Service of the Frontend microservice is the entrypoint of the application and needs to be accessible from outside of the cluster. So we chose the type NodePort and specified an additional port (`nodePort`) which will be accessible from the outside.

The last remaining Deployment and Service is the one for the Redis cache:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cart
spec:
  selector:
    matchLabels:
      app: redis-cart
  template:
    metadata:
      labels:
        app: redis-cart
    spec:
      containers:
      - name: redis
        image: redis:alpine
        ports:
        - containerPort: 6379
        volumeMounts:
        - mountPath: /data
          name: redis-data
      volumes:
      - name: redis-data
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: redis-cart
spec:
  type: ClusterIP
  selector:
    app: redis-cart
  ports:
  - name: redis
    port: 6379
    targetPort: 6379
```

Persistent volume types exist beyond the lifetime of a Pod. But there are also ephemeral volume types that are destroyed by K8s when the Pod using the volume does no longer exist. The volume type `emptyDir` is such an ephemeral volume type. It is created when a Pod referencing this volume is assigned to a node. It exists as long as the Pod is running on that node. When the Pod dies and is re-scheduled on another node, it starts with a new empty directory. (However when a container inside the Pod crashes, the data in the volume will not be lost.)

### Deploy Microservices into Remote K8s Cluster
We are going to deploy these microservices into a remote K8s cluster running on three Linode servers/nodes. So login to your Linode account, open the Kubernetes page and press the "Create Cluster" button. Enter the "Cluster Label" 'online-shop-microservices', select the region 'Frankfurt, DE (eu-central)' and the latest Kubernetes version (1.26). In the "Add Node Pools" section select the "Shared CPU" tab and add three 'Linode 2 GB' servers. Press the "Create Cluster" button. Download the 'online-shop-microservices-kubeconfig.yaml' file.

On your local machine restrict the file permissions for this downloaded file and set the environment variable `KUBECONFIG` to the path of the file:
```sh
chmod 400 </absolute/path/to/online-shop-microservices-kubeconfig.yaml>
export KUBECONFIG=</absolute/path/to/online-shop-microservices-kubeconfig.yaml>`

# test the connection
kubectl get nodes
# =>
# NAME                            STATUS   ROLES    AGE   VERSION
# lke106346-158982-6452c20f5fbc   Ready    <none>   77s   v1.26.3
# lke106346-158982-6452c20fbf34   Ready    <none>   46s   v1.26.3
# lke106346-158982-6452c2101d2f   Ready    <none>   59s   v1.26.3
```

Now create a namespace called `microservices` and deploy all the microservices into this namespace by applying the config.yaml file:
```sh
kubectl create namespace microservices
kubectl apply -f config.yaml -n microservices
# =>
# deployment.apps/emailservice created
# service/emailservice created
# deployment.apps/recommendationservice created
# service/recommendationservice created
# deployment.apps/paymentservice created
# service/paymentservice created
# deployment.apps/productcatalogservice created
# service/productcatalogservice created
# deployment.apps/currencyservice created
# service/currencyservice created
# deployment.apps/shippingservice created
# service/shippingservice created
# deployment.apps/adservice created
# service/adservice created
# deployment.apps/cartservice created
# service/cartservice created
# deployment.apps/checkoutservice created
# service/checkoutservice created
# deployment.apps/frontend created
# service/frontend created
# deployment.apps/redis-cart created
# service/redis-cart created

kubectl get pods -n microservices -w
# =>
# NAME                                     READY   STATUS    RESTARTS   AGE
# adservice-6495d7f86-b8t5t                1/1     Running   0          18s
# cartservice-7c99bd7945-vsdzk             1/1     Running   0          17s
# checkoutservice-b4746cd88-rf5zr          1/1     Running   0          17s
# currencyservice-69b8c58656-742zt         1/1     Running   0          19s
# emailservice-67785f9598-znvgf            1/1     Running   0          20s
# frontend-8f4b9777d-wvmtg                 1/1     Running   0          16s
# paymentservice-6fbc8967d-4r98t           1/1     Running   0          19s
# productcatalogservice-845969555d-h78kn   1/1     Running   0          19s
# recommendationservice-54dcb96dfd-gmd2q   1/1     Running   0          20s
# redis-cart-846556db8f-l7whs              1/1     Running   0          16s
# shippingservice-5459f64756-87mwl         1/1     Running   0          18s

kubectl get services -n microservices
# =>
# NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
# adservice               ClusterIP   10.128.76.151    <none>        9555/TCP       94s
# cartservice             ClusterIP   10.128.64.220    <none>        7070/TCP       93s
# checkoutservice         ClusterIP   10.128.108.84    <none>        5050/TCP       93s
# currencyservice         ClusterIP   10.128.222.36    <none>        7000/TCP       95s
# emailservice            ClusterIP   10.128.28.254    <none>        5000/TCP       96s
# frontend                NodePort    10.128.55.162    <none>        80:30007/TCP   92s
# kubernetes              ClusterIP   10.128.0.1       <none>        443/TCP        142m
# paymentservice          ClusterIP   10.128.192.77    <none>        50051/TCP      95s
# productcatalogservice   ClusterIP   10.128.248.40    <none>        3550/TCP       95s
# recommendationservice   ClusterIP   10.128.244.173   <none>        8080/TCP       96s
# redis-cart              ClusterIP   10.128.55.57     <none>        6379/TCP       92s
# shippingservice         ClusterIP   10.128.97.81     <none>        50051/TCP      94s
```

Services of type NodePort don't get an external IP address. We would have to use a Service of type LoadBalancer for this. But NodePort services are accessible via the IP address of any node of the K8s cluster and the nodePort port of the Service. So login to your Linode management web-console and get the IP address of one of the three nodes. Then open a new browser tab and navigate to `http://<linode-node-ip>:30007` to see the microservices application "boutique" in action.

</details>

*****

<details>
<summary>Video: 23 - Production & Security Best Practices</summary>
<br />

The configuration file created in the last video can and should be improved with several K8s best practices (#1 - #6).

### #1: Specify a pinned version on each container image
Why? Otherwise, the latest version is fetched, which makes it unpredictable and intransparent as to which versions are deployed in the cluster.

All the microservices of the sample application have the same image version. The current version is v0.6.0, so we add this version to each image name, e.g. `gcr.io/google-samples/microservices-demo/frontend` becomes `gcr.io/google-samples/microservices-demo/frontend:v0.6.0`.

### #2: Configure a liveness probe on each container
Why? K8s knows the pod state, not the application state. Sometimes the pod is running, but the container inside crashed. With a liveness probe we can let K8s know when it needs to restart the container.

All the microservices of the sample application already provide a command that can be executed to check the liveness of the microservice. So we just have to tell K8s to call this command for the liveness probe of the container.

Add the following snipped to each container configuration (replace the port number to match the port of each container) except for the 'redis' and 'frontend' container:
```yaml
livenessProbe:
  periodSeconds: 5
  exec:
    command: ["/bin/grpc_health_probe", "-addr=:8080"]
```

There are two other ways of specifying a liveness probe. The first one is providing a REST endpoint (used by the 'frontend' container):
```yaml
livenessProbe:
  periodSeconds: 5
  httpGet:
    path: /health
    port: 8080
```

The second one is simply trying to establish a TCP connection (used by the 'redis' container):
```yaml
livenessProbe:
  periodSeconds: 5
  tcpSocket:
    port: 6379
```

Besides the `periodSeconds` attribute specifying the interval between probes, there is also an `initialDelaySeconds` attribute specifying the initial delay before the first probe is triggered. This may be useful if you know the application will always take some time to start up.

### #3: Configure a readiness probe on each container
Why? Let K8s know if the application is ready to receive traffic. As long as the container is not yet ready, K8s won't trigger any liveness probes on it.

For the microservices of the demo application we use the exact same command used for the liveness probe to add a readiness probe to each container (except for the 'frontend' and 'redis' container):
```yaml
readinessProbe:
  periodSeconds: 5
  exec:
    command: ["/bin/grpc_health_probe", "-addr=:8080"]
```

The readiness probe for the 'frontend' container looks like this:
```yaml
readinessProbe:
  initialDelaySeconds: 10
  httpGet:
    path: "/_healthz"
    port: 8080
    httpHeaders:
    - name: "Cookie"
      value: "shop_session-id=x-readiness-probe"
```

The readiness probe for the 'redis' container looks like this:
```yaml
readinessProbe:
  periodSeconds: 5
  tcpSocket:
    port: 6379
```

### #4: Configure resource limits & requests for each container
Why? To make sure 1 buggy container doesn't eat up all resources, breaking the cluster.

There are two types of resources: CPU and Memory. "requests" is what the container is guaranteed to get. The K8s scheduler has to consider the specified requests to figure out on which node to run new Pods. CPU resources are defined in millicores, Memory resources are defined in Mebibytes (1 Mebibyte = 1024 * 1024 Bytes).

To specify reasonable values for containers without any special requirements, add the following snipped to the container specification:
```yaml
resources:
  requests:
    cpu: 100m
    memory: 64Mi
```

Note that if you specify values bigger than your biggest node, then your Pod will never be scheduled.

To prevent a container from consuming all the available CPU and/or Memory, you should also specify a limit for the container above which it is not allowed to consume the resources. If a container requests more resources than its limits allow it to consume, K8s will stop the Pod of the container and re-schedule it. Limits are defined at the same level as requests:
```yaml
resources:
  requests:
    cpu: 100m
    memory: 64Mi
  limits:
    cpu: 200m
    memory: 128Mi
```

Set these values on every container except for the following:
- the recommendationservice container needs memory of 220Mi/450Mi
- the cartservice container needs CPU of 200m/300m
- the redis container needs CPU of 70m/125m and memory of 200Mi/256Mi
- the adservice container needs CPU of 200m/300m and memory of 180Mi/300Mi

### #5: Don't use NodePort in production
Why? NodePort opens the port on every Worker Node thus exposing them directly and creating multiple points of entry to secure. Better alternatives are LoadBalancer or Ingresss (only one entrypoint to the application), ideally running on a separate Node outside the cluster.

So replace the `NodePort` service type of the frontend service with `LoadBalancer` and remove the `nodePort` attribute.

### #6: Always deploy more than 1 replica for for each application
Why? To make sure your application is always available, no downtime for users!

If you do not specify the number of replicas, the default value of 1 is applied. Add `replicas: 2` to the specification of each deployment:
```yaml
spec:
  replicas: 2
```

****

The following best practices (#7 - #9) are not related to the configuration files but to Kubernetes in general.

### #7: Always have more than 1 Worker Node
Why? Avoid single point of failure with just 1 Node. The replicas of a Pod should ideally run on different Nodes. This allows you to reboot a server (e.g. after a software update or other maintenance tasks) without downtime of the application.

### #8: Label all your K8s resources
Why? Have an identifier for your components, for example to group pods and reference in services. Lables are just key-value pairs and may be technical labels or business labels. 


### #9: Use namespaces to group your resources
Why? For example to organize resources and to define access rights based on namespaces.

****

And finally there are three security related best practices.

### #10: Ensure Images are free of vulnerabilities
Why? Third-party libraries or base images can have known vulnerabilities. You can do manual vulnerability scans or automated scans in CI/CD pipeline.


### #11: No root access for containers
Why? With root access containers have access to host-level resources. Much more damage possible, if container gets hacked!

So don't do this:
```yaml
spec:
  containers:
  - name: app
    image: ...
    securityContext:
      privileged: true
```

If you are using 3rd party images, check whether their containers are running with root privileges.

### #12: Keep K8s version up to date
Why & How? Latest versions include patches to previous security issues etc. Upgrade with zero downtime by having multiple nodes and pod replicas on different nodes and then updating them one node after the other.

</details>

*****

<details>
<summary>Video: 24 - Demo project: Create Helm Chart for Microservices</summary>
<br />

We deployed 10 different microservices, but their Deployment and Service configurations are very similar. An ideal case for the templating feature of Helm which allows to define a blueprint for all the Deployments and Services and only set the specific values for the individual microservices.

Let's create a Helm Chart for the deployment of the microservices.

```sh
# create the basic file/directory structure
helm create microservice
```

Remove everything inside the `microservice/templates/` folder except the files `deployment.yaml` and `service.yaml`. Delete the content of these two remaining files because we're going to writer our templates from scratch. Also delete the content of the `microservice/values.yaml` file.

### Create Basic Template File
In a Helm Chart template file placeholder variables can be defined like this:
```yaml
{{ .Values.varName }}
```

'Values' is a built-in object which contains the values passed into the template from 3 sources:
- the values.yaml file in the chart
- a user-supplied file passed to the helm command with the -f option
- parameters passed to the helm command with the --set option

There are other built-in objects like 'Release', 'Files' etc. For more information read the [documentation](https://helm.sh/docs/chart_template_guide/builtin_objects/).

The values may be defined in a flat or nested way:
```yaml
appName: my-app # referenced as {{ .Values.appName }}
appReplicas: 1  # referenced as {{ .Values.appReplicas }}

app:
  name: my-app # referenced as {{ .Values.app.name }}
  replicas: 1  # referenced as {{ .Values.app.replicas }}
```

Helm best practices recommend to use the flat structure. It is easier for Chart users to set/override the values using the --set option.

### Dynamic Environment Variables
The amount of environment variables is not the same for all containers. So we have to define a list of variable names and values in the values.yaml file. And in the template we must be able to iterate over this list.

For this purpose (iterating over a list of values) there is a built-in function called `range` available. And because the values of environment variables are always strings (even if it's a port number), we have to quote the values. This can be done using another built-in function called `quote`. This function can also be pipelined to the value to be quoted, so `{{ quote .value }}` and `{{ .value | quote }}` are equivalent.

The `env` section in the template finally looks like this:
```yaml
env:
{{- range .Values.containerEnvVars }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}
```

The final `deployment.yaml` template will look like this:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
spec:
  replicas: {{ .Values.appReplicas }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      containers:
      - name: {{ .Values.appName }}
        image: {{ .Values.appImage }}:{{ .Values.appVersion }}
        ports:
        - containerPort: {{ .Values.containerPort }}
        env:
        {{- range .Values.containerEnvVars}}
        - name: {{ .name }}
          value: {{ .value | quote }}
        {{- end}}
```

And the `service.yaml` file will look like this:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
spec:
  type: {{ .Values.serviceType }}
  selector:
    app: {{ .Values.appName }}
  ports:
  - protocol: TCP
    port: {{ .Values.servicePort }}
    targetPort: {{ .Values.containerPort }}
```

### Set the Values
In the `values.yaml` file we can provide default values for the template variables. In our case this could look like this:
```yaml
appName: servicename
appImage: gcr.io/google-samples/microservices-demo/servicename
appVersion: v.0.0.0
appReplicas: 1
containerPort: 8080
containerEnvVars:
- name: ENV_VAR_ONE
  value: "valueone"
- name: ENV_VAR_TWO
  value: "valuetwo"  

serviceType: ClusterIP
servicePort: 8080
```
We are going to override these values for each individual microservice. Let's do it for the first microservice, the emailservice. Create a file called `email-service-values.yaml` outside of the `microservice` folder and add the following content:
```yaml
appName: emailservice
appImage: gcr.io/google-samples/microservices-demo/emailservice
appVersion: v0.6.0
appReplicas: 2
containerPort: 8080
containerEnvVars:
- name: PORT
  value: "8080"

servicePort: 5000
```

Let's check whether the first microservice configuration files will be generated correctly. Go outside of the `microservice` directory and execute the following command:
```sh
helm template -f email-service-values.yaml microservice
```

This command won't create any files. It just prints out what would be sent to Kubernetes when `helm install` would be executed.

There is also a `helm lint` command that examines a chart for possible issues. Issues reported as ERROR will cause the chart to fail on installation. Issues reported as WARNING just break the conventions or recommendations.
```sh
helm lint -f email-service-values.yaml microservice
# ==> Linting microservice
# [INFO] Chart.yaml: icon is recommended

# 1 chart(s) linted, 0 chart(s) failed
```

### Deploy Email Service
To deploy the configured emailservice into the Kubernetes cluster execute the following command:
```sh
helm install -f email-service-values.yaml emailservice microservice
#    ------- ---------------------------- ------------ ------------
#    install        override values          release       chart
#    a chart          from a file             name         name

helm ls
# =>
# NAME        	NAMESPACE	REVISION	UPDATED                              	STATUS  	CHART             	APP VERSION
# emailservice	default  	1       	2023-05-05 23:25:05.133891 +0200 CEST	deployed	microservice-0.1.0	1.16.0
```

The chart version and app version values are taken from the `Chart.yaml` file which we didn't adjust yet.

### Create Values File for all Microservices
To deploy all the microservices using the same Helm Chart we first have to create a service-specific values.yaml file for each microservice (except for the 'redis' microservice). We put all these service-specific values files into a folder called `values` (just for keeping the workspace clean).

### Create Redis Helm Chart
Because the redis microservice uses a 3rd party image and thus probably has a lifecycle different from the one of the other microservices, we want to keep it separated from our own microservices. So let's create a Helm Chart for the redis microservice only.

We create a `charts` directory, move the microservice chart into it and create a new `redis` chart:
```sh
mkdir charts
mv microservice charts
cd charts
helm create redis
```

Cleanup all the auto-generated files within the redis chart folder (as we did before for the microservice chart), and add the following content to the deployment.yaml file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.appName }}
spec:
  replicas: {{ .Values.appReplicas }}
  selector:
    matchLabels:
      app: {{ .Values.appName }}
  template:
    metadata:
      labels:
        app: {{ .Values.appName }}
    spec:
      containers:
      - name: {{ .Values.appName }}
        image: {{ .Values.appImage }}:{{ .Values.appVersion }}
        ports:
        - containerPort: {{ .Values.containerPort }}
        volumeMounts:
        - name: {{ .Values.volumeName }}
          mountPath: {{ .Values.containerMountPath }}
      volumes: 
      - name: {{ .Values.volumeName }}
        emptyDir: {}
```

Then add the follwing content to the service.yaml file:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.appName }}
spec:
  type: ClusterIP
  selector:
    app: {{ .Values.appName }}
  ports:
  - protocol: TCP
    port: {{ .Values.servicePort }}
    targetPort: {{ .Values.containerPort }}
```

Add the following default values to the values.yaml file:
```yaml
appName: redis
appImage: redis
appVersion: alpine
appReplicas: 1
containerPort: 6379
volumeName: redis-data
containerMountPath: /data

servicePort: 6379
```

And finally add a `redis-values.yaml` file with the following content to the `values` folder:
```yaml
appName: redis-cart
appReplicas: 2
```

</details>

*****