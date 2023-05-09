## Exercises for Module 10 "Container Orchestration with Kubernetes"
<br />

Your _bootcamp-java-mysql application (from Docker module exercises)_ is running with docker-compose on a server. This application is used often internally and by your company clients too. You noticed that the server isn't very stable: Often a database container dies or the application itself, or docker daemon must be restarted. At this time people can't access the app!

So when this happens, the users write you that the app is down and ask you to fix it. You ssh into the server, restart containers with docker-compose and containers start again.

But this is an annoying work, plus it doesn't look good for your company that your clients often can't access the app. So you want to make your application **more reliable and highly available**. You want to **replicate** both database and the app, so if one container goes down, there is always a backup. Also you don't want to rely on a single server, but have multiple, in case 1 whole server goes down or gets rebooted etc.


So you look into different solutions and decide to use a container orchestration tool Kubernetes to solve the issue. For now you want to configure it and deploy your application manually, since it's a new tool and want to try it out manually before automating.

<details>
<summary>Exercise 1: Create a Kubernetes cluster</summary>
<br />

**Tasks:**
- Create a Kubernetes cluster (Minikube or LKE)

**Steps to solve the tasks:**

**Minikube**\
On a Mac with M2 processor the easiest way to install minikube is using the `homebrew` package manager:
```sh
brew update
brew install minikube
minikube start --driver docker
minikube status
```
During `minikube` installation `kubectl` gets automatically installed too (as a dependency).

**LKE**\
Login to your [Linode account](https://cloud.linode.com/), select "Kubernetes" in the menu on the left and press the blue "Create Cluster" button. Enter a cluster name (e.g. 'devops-bootcamp'), choose a region close to you (e.g. 'Frankfurt, DE (eu-central)') and select the latest Kubernetes version (e.g. 1.26). In the "Add Node Pools" section select the "Shared CPU" tab and add 2 "Linode 4 GB" nodes to the cart. Check the "I have read..." disclaimer and press the "Create Cluster" button.

On the dashboard you can see the two worker nodes (Linodes). Wait until both are up and running.

In the Kubernetes section at the top you can download a 'devops-bootcamp-kubeconfig.yaml' file with the credentials and certificates you need to connect to the K8s cluster. Download it and set the environment variable KUBECONFIG on your local machine to this file:
```sh
export KUBECONFIG=~/Downloads/devops-bootcamp-kubeconfig.yaml

# now kubectl commands will be connected with the linode cluster
kubectl get nodes
# =>
# NAME                            STATUS   ROLES    AGE   VERSION
# lke104424-156177-6445973ec1e1   Ready    <none>   19m   v1.26.3
# lke104424-156177-6445973f23f0   Ready    <none>   19m   v1.26.3
```

</details>

******

<details>
<summary>Exercise 2: Deploy Mysql with 3 replicas</summary>
<br />

**Tasks:**

First of all, you want to deploy the mysql database.
- Deploy Mysql database with 3 replicas and volumes for data persistence 

To simplify the process you can use Helm for that.

**Steps to solve the tasks:**

If you haven't installed Helm yet, [install it now](https://helm.sh/docs/intro/install/). On a Mac, the easiest way to install Helm is to execute
```sh
brew update
brew install helm
```

Google for "Helm Charts Mysql". You should find the charts maintained by [Bitnami](https://bitnami.com/stack/mysql/helm). Execute the following commands:
```sh
# add the bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami

# search for mysql charts in this repo
helm search repo bitnami/mysql
# =>
# NAME            CHART VERSION	  APP VERSION   DESCRIPTION                                       
# bitnami/mysql   9.8.2           8.0.33     	MySQL is a fast, reliable, scalable, and easy t...
```

To see the parameters of the chart, open the browser and navigate to `https://github.com/bitnami/charts/tree/main/bitnami/mysql`. You'll find that there are parameters `architecture`, `auth.rootPassword`, `secondary.replicaCount`, `secondary.persistence.storageClass` (among many others). To override these parameters for deployment on a **Minikube** cluster create a file called `mysql-chart-values-minikube.yaml` with the following content:
```yaml
architecture: replication
auth:
  rootPassword: secret-root-pass
  database: my-app-db
  username: my-user
  password: my-pass

secondary:
  # 1 primary and 2 secondary replicas
  replicaCount: 2
  persistence:
    storageClass: standard
```

For deployment on **Linode LKE** create a file called `mysql-chart-values-lke.yaml` with the following content:
```yaml
architecture: replication
auth:
  rootPassword: secret-root-pass
  database: my-app-db
  username: my-user
  password: my-pass

# enable init container that changes the owner and group of the persistent volume mountpoint to runAsUser:fsGroup
volumePermissions:
  enabled: true

secondary:
  # 1 primary and 2 secondary replicas
  replicaCount: 2
  persistence:
    accessModes: ["ReadWriteOnce"]
    # storage class for LKE volumes
    storageClass: linode-block-storage
```

To install the chart in the local **Minikube** cluster execute the following commands:
```sh
helm install -f mysql-chart-values-minikube.yaml my-release bitnami/mysql

kubectl get all
# NAME                               READY   STATUS    RESTARTS   AGE
# pod/my-release-mysql-primary-0     1/1     Running   0          4m48s
# pod/my-release-mysql-secondary-0   1/1     Running   0          4m48s
# pod/my-release-mysql-secondary-1   1/1     Running   0          3m16s
# 
# NAME                                          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
# service/kubernetes                            ClusterIP   10.96.0.1      <none>        443/TCP    22d
# service/my-release-mysql-primary              ClusterIP   10.97.202.97   <none>        3306/TCP   4m48s
# service/my-release-mysql-primary-headless     ClusterIP   None           <none>        3306/TCP   4m48s
# service/my-release-mysql-secondary            ClusterIP   10.111.6.1     <none>        3306/TCP   4m48s
# service/my-release-mysql-secondary-headless   ClusterIP   None           <none>        3306/TCP   4m48s
# 
# NAME                                          READY   AGE
# statefulset.apps/my-release-mysql-primary     1/1     4m48s
# statefulset.apps/my-release-mysql-secondary   2/2     4m48s
```

To install the chart in a **Linode LKE** cluster execute the following commands:
```sh
helm install -f mysql-chart-values-lke.yaml my-release bitnami/mysql
kubectl get statefulset --watch
```

</details>

******

<details>
<summary>Exercise 3: Deploy your Java Application with 3 replicas</summary>
<br />

**Tasks:**

Now you want to
- deploy your Java application with 3 replicas.

With docker-compose, you were setting env_vars on server. In K8s there are own components for that, so
- create ConfigMap and Secret with the values and reference them in the application deployment config file.

**Steps to solve the tasks:**

**Step 1:** Push docker image of java mysql app to private registry if necessary\
Make sure the image of the [bootcamp-java-mysql](https://github.com/fsiegrist/devops-bootcamp-07-docker/tree/main/bootcamp-java-mysql) app from the exercises of module 7 is available in the private repository on DockerHub. If it is not go to the application folder, 
create a jar file executing
```sh
./gradlew build
```
create a docker image executing 
```sh
docker build -t fsiegrist/fesi-repo:bootcamp-java-mysql-project-1.1-SNAPSHOT .
```
and push the image to remote private docker registry executing
```sh
docker login
docker push fsiegrist/fesi-repo:bootcamp-java-mysql-project-1.1-SNAPSHOT
```

**Step 2:** Create a 'my-registry-key' Secret to pull the image from the private repository on  DockerHub
```sh
kubectl create secret docker-registry my-registry-key \
  --docker-server=docker.io \
  --docker-username=fsiegrist \
  --docker-password=<my-docker-hub-pwd>
```

**Step 3:** Create the required K8s component configuration files

Create a ConfigMap configuration file with the folowing content:

_db-config.yaml_
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  db_server: my-release-mysql-primary # kubectl get services
```

Create a Secret configuration file with the folowing content:

_db-secret.yaml_
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  # echo -n 'my-user' | base64 (see mysql-chart-values-minikube.yaml)
  db_user: bXktdXNlcg== 
  # echo -n 'my-pass' | base64 (see mysql-chart-values-minikube.yaml)
  db_pwd: bXktcGFzcw==
  # echo -n 'my-app-db' | base64 (see mysql-chart-values-minikube.yaml)
  db_name: bXktYXBwLWRi
  # echo -n 'secret-root-pass' | base64 (see mysql-chart-values-minikube.yaml)
  db_root_pwd: c2VjcmV0LXJvb3QtcGFzcw==
```

Create a Deployment and Service configuration file with the folowing content:

_java-mysql-app.yaml_
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-mysql-app-deployment
  labels:
    app: java-mysql-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: java-mysql-app
  template:
    metadata:
      labels:
        app: java-mysql-app
    spec:
      imagePullSecrets:
      - name: my-registry-key
      containers:
      - name: javamysqlapp
        image: fsiegrist/fesi-repo:bootcamp-java-mysql-project-1.1-SNAPSHOT
        ports:
        - containerPort: 8080
        env:
         - name: DB_USER
           valueFrom:
             secretKeyRef:
               name: db-secret
               key: db_user
         - name: DB_PWD
           valueFrom:
             secretKeyRef:
               name: db-secret
               key: db_pwd
         - name: DB_NAME
           valueFrom:
             secretKeyRef:
               name: db-secret
               key: db_name
         - name: DB_SERVER
           valueFrom:
             configMapKeyRef:
              name: db-config
              key: db_server
---
apiVersion: v1
kind: Service
metadata:
  name: java-mysql-app-service
spec:
  selector:
    app: java-mysql-app
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
```

**Step 4:** Apply the configurations to the K8s cluster\
```sh
kubectl apply -f db-config.yaml
kubectl apply -f db-secret.yaml
kubectl apply -f java-mysql-app.yaml

kubectl get pods -l app=java-mysql-app
# NAME                                         READY   STATUS    RESTARTS   AGE
# java-mysql-app-deployment-574674d7d9-86wbs   1/1     Running   0          8m16s
# java-mysql-app-deployment-574674d7d9-vr2l8   1/1     Running   0          8m16s
# java-mysql-app-deployment-574674d7d9-x4qgc   1/1     Running   0          8m16s
```

**Step 5 (optional):** Create a port-forwarding to access the application
```sh
kubectl port-forward svc/java-mysql-app-service 8080:8080
```

Open the browser and navigate to [localhost:8080](http://localhost:8080) to access the running application.

Note: Because of CORS restrictions API calls triggered by the application are blocked. To avoid this, we have to setup ingress (see exercises 5 and 6).

</details>

******

<details>
<summary>Exercise 4: Deploy phpmyadmin</summary>
<br />

**Tasks:**

As a next step you
- deploy phpmyadmin to access Mysql UI.

For this deployment you just need 1 replica, since this is only for your own use, so it doesn't have to be High Availability. A simple deployment.yaml file and internal service will be enough.

**Steps to solve the tasks:**

**Step 1:** Create a Deployment and Service configuration file for phpmyadmin

_phpmyadmin.yaml_
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: phpmyadmin
  labels:
    app: phpmyadmin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: phpmyadmin
  template:
    metadata:
      labels:
        app: phpmyadmin
    spec:
      containers:
        - name: phpmyadmin
          image: phpmyadmin/phpmyadmin:5
          ports:
            - containerPort: 80
              protocol: TCP
          env:
            - name: PMA_HOST
              valueFrom:
                configMapKeyRef:
                  name: db-config
                  key: db_server
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-secret
                  key: db_root_pwd
  
---
apiVersion: v1
kind: Service
metadata:
  name: phpmyadmin-service
spec:
  selector:
    app: phpmyadmin
  ports:
  - protocol: TCP
    port: 8081
    targetPort: 80
```

**Step 2:** Apply it to the cluster
```sh
kubectl apply -f phpmyadmin.yaml

kubectl get pods -l app=phpmyadmin
# NAME                          READY   STATUS    RESTARTS   AGE
# phpmyadmin-794dd6c7fb-xxlrw   1/1     Running   0          3m40s
```

</details>

******

Now your application setup is running in the cluster, but you still need a proper way to access the application. Also, you don't want users to access the application using the IP address and instead use a domain name. For that, you want to install Ingress controller in the cluster and configure ingress access for your application.


<details>
<summary>Exercise 5: Deploy Ingress Controller</summary>
<br />

**Tasks:**
- Deploy Ingress Controller in the cluster - using Helm

**Steps to solve the tasks:**

</details>

******

<details>
<summary>Exercise 6: Create Ingress rule</summary>
<br />

**Tasks:**
- Create Ingress rule for your application access

**Steps to solve the tasks:**

</details>

******

<details>
<summary>Exercise 7: Port-forward for phpmyadmin</summary>
<br />

**Tasks:**

However, you don't want to expose the phpmyadmin for security reasons. So you configure port-forwarding for the service to access on localhost, whenever you need it.
- Configure port-forwarding for phpmyadmin

**Steps to solve the tasks:**

</details>

******

<details>
<summary>Exercise 8: Create Helm Chart for Java App</summary>
<br />

As the final step, you decide to create a helm chart for your Java application where all the configuration files are configurable. You can then tell developers how they can use it by setting all the chart values. This chart will be hosted in its own git repository. 

**Tasks:**
- All config files: service, deployment, ingress, configMap, secret, will be part of the chart
- Create custom values file as an example for developers to use when deploying the application
- Deploy the java application using the chart with helmfile
- Host the chart in its own git repository

**Steps to solve the tasks:**

</details>

******
