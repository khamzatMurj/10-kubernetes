## Demo Project - Install Stateful Service Using Helm

### Topics of the Demo Project
Install a stateful service (MongoDB) on Kubernetes using Helm.

### Technologies Used
- Kubernetes
- Helm
- MongoDB
- Mongo Express
- Linode LKE
- Linux

### Project Description
- Create a managed K8s cluster with Linode Kubernetes Engine
- Deploy replicated MongoDB service in LKE cluster using a Helm chart
- Configure data persistence for MongoDB with Linode’s cloud storage
- Deploy UI client Mongo Express for MongoDB
- Deploy and configure nginx ingress to access the UI application from browser

#### Steps to create a managed K8s cluster with Linode Kubernetes Engine
- Login to your [Linode account](https://cloud.linode.com/), press the blue "Create" button and select "Kubernetes". 
- Enter a cluster name (e.g. 'devops-bootcamp'), choose a region close to you (e.g. 'Frankfurt, DE (eu-central)') and select the latest Kubernetes version (e.g. 1.26). 
- In the "Add Node Pools" section select the "Shared CPU" tab and add 2 "Linode 4 GB" nodes to the cart. Check the "I have read..." disclaimer and press the "Create Cluster" button.
- On the dashboard you can see the two worker nodes (Linodes). Wait until both are up and running.
- In the Kubernetes section at the top you can download a 'devops-bootcamp-kubeconfig.yaml' file with the credentials and certificates you need to connect to the K8s cluster. Download it.

On your local machine set the environment variable KUBECONFIG to this file:
```sh
export KUBECONFIG=</path/to/download-folder>/devops-bootcamp-kubeconfig.yaml

# now kubectl commands will be connected with the linode cluster
kubectl get nodes
# =>
# NAME                            STATUS   ROLES    AGE   VERSION
# lke104424-156177-6445973ec1e1   Ready    <none>   19m   v1.26.3
# lke104424-156177-6445973f23f0   Ready    <none>   19m   v1.26.3
```

#### Steps to deploy a replicated stateful MongoDB service and configure data persistence for MongoDB with Linode’s cloud storage in LKE cluster using a Helm chart
**Step 1:** Install Helm\
If you haven't installed Helm yet, [install it now](https://helm.sh/docs/intro/install/). On a Mac, the easiest way to install Helm is to execute
```sh
brew update
brew install helm
```

**Step 2:** Find suitable Helm Charts\
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

**Step 3:** Override parameters\
To see the parameters of the chart, open the browser and navigate to `https://github.com/bitnami/charts/tree/main/bitnami/mongodb`. You'll find that there are parameters `architecture`, `replicaCount`, `persistence.storageClass` and `auth.rootPassword` (among many others). To override these parameters create a file called `devops-bootcamp-mongodb-values.yaml` with the following content:
```yaml
architecture: replicaset
replicaCount: 3
persistence:
    storageClass: "linode-block-storage"
auth:
    rootPassword: t0p-secret
```

**Step 4:** Install the Chart\
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

**Step 5:** Check volumes have been created\
In the Linode web console you should see that three persistent volumes have been created (one for each replica) and attached to the two worker nodes.

#### Steps to deploy UI client Mongo Express for MongoDB
**Step 1:** Create a K8s configuration file\
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

**Step 2:** Apply the file\
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

#### Steps to deploy and configure nginx ingress to access the UI application from browser
**Step 1:** Install nginx-ingress controller\
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

**Step 2:** Create a K8s configuration file for the Ingress rule\
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

**Step 3:** Apply the file to the cluster
```sh
kubectl apply -f devops-bootcamp-ingress.yaml

# check it
kubectl get ingress
# =>
# NAME            CLASS    HOSTS                                    ADDRESS         PORTS   AGE
# mongo-express   <none>   143-42-222-35.ip.linodeusercontent.com   143.42.222.35   80      59s
```

**Step 4:** Browse the application\
Now you can access the MongoExpress application in your browser navigating to [http://143-42-222-35.ip.linodeusercontent.com](http://143-42-222-35.ip.linodeusercontent.com). Create a new database and collection in it.

#### Steps to remove all the created components (optional)
If you want to remove all the components created via Helm Charts, just execute:
```sh
helm uninstall ingress-nginx
helm uninstall mongodb
```

Note that the PersistentVolumes that have been created when installing the mongodb chart, don't get deleted by `helm uninstall`. This is a security feature, to keep the data in case you want to re-install the database later.