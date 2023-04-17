## Demo Project - Deploy Containers on local Kubernetes Cluster

### Topics of the Demo Project
Deploy MongoDB and Mongo Express into local K8s cluster

### Technologies Used
- Kubernetes
- Docker
- MongoDB
- Mongo Express

### Project Description
- Setup local K8s cluster with Minikube
- Deploy MongoDB and MongoExpress with configuration and credentials extracted into ConfigMap and Secret

#### Steps to setup local K8s cluster with Minikube
On my Mac with M2 processor the easiest way to install minikube is using the `homebrew` package manager:
```sh
brew update
brew install minikube
minikube start --driver docker
minikube status
```
During `minikube` installation `kubectl` gets automatically installed too (as a dependency).

#### Steps to deploy MongoDB and MongoExpress
**Step 1:** Create Secret with username and password for MongoDB\
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

Create the secret in the minikube cluster by applying the configuration file:
```sh
kubectl apply -f mongodb-secret.yaml
```

**Step 2:** Create MongoDB Deployment
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

Create the deployment in the minikube cluster by applying the configuration file:
```sh
kubectl apply -f mongodb.yaml
```

**Step 3:** Create an internal Service for the mongodb Pod\
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

Now re-apply the changes:
```sh
kubectl apply -f mongodb.yaml
```

**Step 4:** Create a ConfigMap containing the MongoDB URL
Create a file called `mongodb-configmap.yaml` with the following content:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongodb-configmap
data:
  database_url: mongodb-service # must match the name of the internal service
```

Create the ConfigMap in the minikube cluster by applying the configuration file:
```sh
kubectl apply -f mongodb-configmap.yaml
```

**Step 5:** Create a MongoExpress Deployment
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

Create the deployment in the minikube cluster by applying the configuration file:
```sh
kubectl apply -f mongoexpress.yaml
```

**Step 6:** Create an external Service for the MongoExpress Pod\
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
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
      nodePort: 30000
```

Now re-apply the changes and list all services:
```sh
kubectl apply -f mongoexpress.yaml
kubectl get services
# =>
#NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
#kubernetes              ClusterIP      10.96.0.1      <none>        443/TCP          23m
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
