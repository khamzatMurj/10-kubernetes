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
brew update
brew install minikube
minikube start --driver docker
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

Kubectl is a CLI tool to interact with your K8s cluster. In order for kubectl to access a K8s cluster, it needs a kubeconfig file, which is created automatically when deploying your minikube cluster. By default, the config file is located at `~/.kube/config`.

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