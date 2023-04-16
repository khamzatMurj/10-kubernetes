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

### Basic kubctl Commands

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

### Kubernetes Configuration File

Kubernetes configuration/manifest files are declarative, i.e. they specify the desired state of a K8s component. Each configuration file has 3 parts:
- metadata
- specification: the attributes of "spec" are specific to the component kind
- status: automatically generated and added by K8s; K8s gets this information from etcd, which holds the current status of any K8s component; if the current status differs from the specified desired status, K8s tries to reach the desired status

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
  template:
    metadata:
      labels:
        app: devops
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
        ports:
        - containerPort: 80
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

</details>

*****
