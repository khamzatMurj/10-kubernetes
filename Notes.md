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