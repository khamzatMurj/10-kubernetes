## Demo Project - Deploy a Microservices Application to a Remote Kubernetes Cluster

### Topics of the Demo Project
Deploy Microservices application in Kubernetes with Production & Security Best Practices

### Technologies Used
- Kubernetes
- Redis
- Linux
- Linode LKE

### Project Description
- Create K8s manifests for Deployments and Services for all microservices of an online shop application
- Deploy the microservices to Linode’s managed Kubernetes cluster

#### Steps to create K8s manifests for Deployments and Services for all microservices of an online shop application

We are going to deploy the microservices application in the GitHub repository [nanuchi/microservices-demo](https://github.com/nanuchi/microservices-demo) into a Linode K8s cluster.

The application is made up of the following microservices:
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

All the microservices are developped by one single team. So we decide to deploy them all into the same namespace.

Since the example microservices application is a fork of a demo application provided by Google, the images are available in a [public Google image registry](https://console.cloud.google.com/gcr/images/google-samples/global/microservices-demo). The currently latest version of the images is `v0.6.0`.

Each microservice container needs an environment variable called `PORT` specifying the port the container is listening on.

And each container provides a command that can be used to check the readiness and liveness of the container:
```sh
/bin/grpc_health_probe -addr=:<container-port>
```

**Step 1:** Create an empty configuration file:
```sh
mkdir online-shop-microservices
cd online-shop-microservices
touch config.yaml
```

**Step 2:** Specify the Deployment and Service for the email microservice\
Add the following content to the `config.yaml` file:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: emailservice
spec:
  replicas: 2
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
        image: gcr.io/google-samples/microservices-demo/emailservice:v0.6.0
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        readinessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:8080"]
        livenessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:8080"]
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
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

**Step 3:** Specify the Deployment and Service for the recommendation microservice\
Since the recommendation microservice is talking to the product-catalog microservice we have to tell it under which hostname and port this service can be accessed. Since all the Services are running in the same namespace, their hostname is just the service name. We are going to name the Service for the product-catalog microservice `productcatalogservice`. It will be listening on port 3550 (attribute `port`). The address of the product-catalog microservice's Service is stored in the environment variable `PRODUCT_CATALOG_SERVICE_ADDR`.

Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recommendationservice
spec:
  replicas: 2
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
        image: gcr.io/google-samples/microservices-demo/recommendationservice:v0.6.0
        ports:
        - containerPort: 8080
        env:
        - name: PORT
          value: "8080"
        - name: PRODUCT_CATALOG_SERVICE_ADDR
          value: "productcatalogservice:3550"
        readinessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:8080"]
        livenessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:8080"]
        resources:
          requests:
            cpu: 100m
            memory: 220Mi
          limits:
            cpu: 200m
            memory: 450Mi
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

**Step 4:** Specify the Deployment and Service for the payment microservice\
Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: paymentservice
spec:
  replicas: 2
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
        image: gcr.io/google-samples/microservices-demo/paymentservice:v0.6.0
        ports:
        - containerPort: 50051
        env:
        - name: PORT
          value: "50051"
        readinessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:50051"]
        livenessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:50051"]
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
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
```

**Step 5:** Specify the Deployment and Service for the product-catalog microservice\
Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: productcatalogservice
spec:
  replicas: 2
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
        image: gcr.io/google-samples/microservices-demo/productcatalogservice:v0.6.0
        ports:
        - containerPort: 3550
        env:
        - name: PORT
          value: "3550"
        readinessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:3550"]
        livenessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:3550"]
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
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
```

**Step 6:** Specify the Deployment and Service for the currency microservice\
Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: currencyservice
spec:
  replicas: 2
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
        image: gcr.io/google-samples/microservices-demo/currencyservice:v0.6.0
        ports:
        - containerPort: 7000
        env:
        - name: PORT
          value: "7000"
        readinessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:7000"]
        livenessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:7000"]
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
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
```

**Step 7:** Specify the Deployment and Service for the shipping microservice\
Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shippingservice
spec:
  replicas: 2
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
        image: gcr.io/google-samples/microservices-demo/shippingservice:v0.6.0
        ports:
        - containerPort: 50051
        env:
        - name: PORT
          value: "50051"
        readinessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:50051"]
        livenessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:50051"]
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: 200m
            memory: 128Mi
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
```

**Step 8:** Specify the Deployment and Service for the ad microservice\
The ad-service needs an initial probe delay and more CPU and memory. Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: adservice
spec:
  replicas: 2
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
        image: gcr.io/google-samples/microservices-demo/adservice:v0.6.0
        ports:
        - containerPort: 9555
        env:
        - name: PORT
          value: "9555"
        readinessProbe:
          initialDelaySeconds: 20
          periodSeconds: 15
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:9555"]
        livenessProbe:
          initialDelaySeconds: 20
          periodSeconds: 15
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:9555"]
        resources:
          requests:
            cpu: 200m
            memory: 180Mi
          limits:
            cpu: 300m
            memory: 300Mi
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

**Step 9:** Specify the Deployment and Service for the cart microservice\
The cart-service needs more CPU. It does not define a `PORT` environment variable but a `REDDIS_ADDR` instead. The Redis cart will be configured at the end. Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cartservice
spec:
  replicas: 2
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
        image: gcr.io/google-samples/microservices-demo/cartservice:v0.6.0
        ports:
        - containerPort: 7070
        env:
        - name: REDIS_ADDR
          value: "rediscart:6379"
        readinessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:7070"]
        livenessProbe:
          periodSeconds: 5
          exec:
            command: ["/bin/grpc_health_probe", "-addr=:7070"]
        resources:
          requests:
            cpu: 200m
            memory: 64Mi
          limits:
            cpu: 300m
            memory: 128Mi
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

**Step 10:** Specify the Deployment and Service for the checkout microservice\
The checkout service talks to a lot of other microservices and thus needs an environment variable for each of these services defining the according Service address. Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: checkoutservice
spec:
  replicas: 2
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
          image: gcr.io/google-samples/microservices-demo/checkoutservice:v0.6.0
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
          readinessProbe:
            periodSeconds: 5
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:5050"]
          livenessProbe:
            periodSeconds: 5
            exec:
              command: ["/bin/grpc_health_probe", "-addr=:5050"]
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
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
```

**Step 11:** Specify the Deployment and Service for the frontend microservice\
Like the checkout service the frontend service talks to a lot of other microservices and thus needs an environment variable for each of these services defining the according Service address.

The Service of the Frontend microservice is the entrypoint of the application and needs to be accessible from outside of the cluster. So we choose the type LoadBalancer.

For the readiness and liveness probes we use a REST endpoint provided by the frontend microservice. The endpoint is available under the path `/_healthz` on port 8080. A cookie named `shop_session-id` specifies whether it's a readiness probe or a liveness probe.

Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
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
          image: gcr.io/google-samples/microservices-demo/frontend:v0.6.0
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
          readinessProbe:
            initialDelaySeconds: 10
            httpGet:
              path: "/_healthz"
              port: 8080
              httpHeaders:
              - name: "Cookie"
                value: "shop_session-id=x-readiness-probe"
          livenessProbe:
            initialDelaySeconds: 10
            httpGet:
              path: "/_healthz"
              port: 8080
              httpHeaders:
              - name: "Cookie"
                value: "shop_session-id=x-liveness-probe"
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
  - name: http
    port: 80
    targetPort: 8080
```

**Step 12:** Specify the Deployment and Service for the redis microservice\
Because this service stores the data of the online shop, we want to have only one replica. Otherwhise we would have to setup synchronization between two data stores. The redis service needs less CPU but more memory. The container does not provide a command or REST endpoint to check the readiness or liveness, so we use the `tcpSocket` probe just ckecking the connection. To store the data of this demo application we use the ephemeral volume type `emptyDir` just providing a directory to store the data for the livetime of the Pod. Append the following content to the `config.yaml` file:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rediscart
spec:
  replicas: 1
  selector:
    matchLabels:
      app: rediscart
  template:
    metadata:
      labels:
        app: rediscart
    spec:
      containers:
      - name: redis
        image: redis:alpine
        ports:
        - containerPort: 6379
        readinessProbe:
          periodSeconds: 5
          tcpSocket:
            port: 6379
        livenessProbe:
          periodSeconds: 5
          tcpSocket:
            port: 6379
        resources:
          requests:
            cpu: 70m
            memory: 200Mi
          limits:
            cpu: 125m
            memory: 256Mi
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
  name: rediscart
spec:
  type: ClusterIP
  selector:
    app: rediscart
  ports:
  - name: redis
    port: 6379
    targetPort: 6379
```

#### Steps to deploy the microservices to Linode’s managed Kubernetes cluster
**Step 1:** Create Linode Kubernetes cluster with three nodes\
We login to our Linode account, open the Kubernetes page and press the "Create Cluster" button. We enter the "Cluster Label" 'online-shop-microservices', select the region 'Frankfurt, DE (eu-central)' and the latest Kubernetes version (1.26). In the "Add Node Pools" section we select the "Shared CPU" tab and add three 'Linode 4 GB' servers. We press the "Create Cluster" button and download the 'online-shop-microservices-kubeconfig.yaml' file.

**Step 2:** Configure local machine
On our local machine we restrict the file permissions for the downloaded file and set the environment variable `KUBECONFIG` to the path of the file:
```sh
# we are still in the online-shop-microservices directory
mv ~/Downloads/online-shop-microservices-kubeconfig.yaml .
chmod 400 ./online-shop-microservices-kubeconfig.yaml
export KUBECONFIG=$(pwd)/online-shop-microservices-kubeconfig.yaml

# test the connection
kubectl get nodes
# =>
# NAME                            STATUS   ROLES    AGE   VERSION
# lke106346-158982-6452c20f5fbc   Ready    <none>   77s   v1.26.3
# lke106346-158982-6452c20fbf34   Ready    <none>   46s   v1.26.3
# lke106346-158982-6452c2101d2f   Ready    <none>   59s   v1.26.3
```

**Step 3:** Create a namespace and deploy the microservices\
We execute the following commands:
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

kubectl get pods -n microservices
# =>
# NAME                                     READY   STATUS             RESTARTS        AGE
# adservice-6495d7f86-b8t5t                1/1     Running            1 (6m9s ago)    6m30s
# cartservice-7c99bd7945-vsdzk             1/1     Running            0               6m30s
# checkoutservice-b4746cd88-rf5zr          1/1     Running            0               3m9s
# currencyservice-69b8c58656-742zt         0/1     CrashLoopBackOff   5 (2m44s ago)   6m30s
# emailservice-67785f9598-znvgf            1/1     Running            0               7m27s
# frontend-8f4b9777d-wvmtg                 1/1     Running            0               3m9s
# paymentservice-6fbc8967d-4r98t           0/1     CrashLoopBackOff   4 (82s ago)     3m9s
# productcatalogservice-845969555d-h78kn   1/1     Running            0               6m30s
# recommendationservice-54dcb96dfd-gmd2q   1/1     Running            0               7m27s
# redis-cart-846556db8f-l7whs              1/1     Running            0               3m8s
# shippingservice-5459f64756-87mwl         1/1     Running            0               6m30s
```

The currencyservice and the paymentservice seem to have problems. Let's check the logs of the currencyservice Pod:
```sh
kubectl logs currencyservice-69b8c58656-742zt -n microservices
# =>
# Profiler enabled.
# Tracing disabled.
# ...
# /usr/src/app/node_modules/@google-cloud/profiler/build/src/index.js:120
#         throw new Error('Project ID must be specified in the configuration');
#               ^
# 
# Error: Project ID must be specified in the configuration
#     at initConfigMetadata (/usr/src/app/node_modules/@google-cloud/profiler/build/src/index.js:120:15)
#     at process.processTicksAndRejections (node:internal/process/task_queues:95:5)
#     at async createProfiler (/usr/src/app/node_modules/@google-cloud/profiler/build/src/index.js:158:26)
#     at async Object.start (/usr/src/app/node_modules/@google-cloud/profiler/build/src/index.js:182:22)
```
There seems to be a problem while creating a profiler. The [Source Code](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/src/currencyservice/server.js) shows that the profiler can be disabled by setting an environment variable `DISABLE_PROFILER` to a truthy value.

The paymentservice seems to have the same problem. So we set an additional env variable named `DISABLE_PROFILER` to `"true"` on both services:
```yaml
- name: DISABLE_PROFILER
  value: "true"
```

Now we re-apply the config and check the result:
```sh
kubectl apply -f config.yaml -n microservices

kubectl get pods -n microservices
# =>
# NAME                                     READY   STATUS    RESTARTS      AGE
# adservice-6495d7f86-b8t5t                1/1     Running   1 (38m ago)   39m
# cartservice-7c99bd7945-vsdzk             1/1     Running   0             39m
# checkoutservice-b4746cd88-rf5zr          1/1     Running   0             35m
# currencyservice-65859cc6dd-fqq7t         1/1     Running   0             24s
# emailservice-67785f9598-znvgf            1/1     Running   0             40m
# frontend-8f4b9777d-wvmtg                 1/1     Running   0             35m
# paymentservice-7f84986fd6-kthfv          1/1     Running   0             25s
# productcatalogservice-845969555d-h78kn   1/1     Running   0             39m
# recommendationservice-54dcb96dfd-gmd2q   1/1     Running   0             40m
# redis-cart-846556db8f-l7whs              1/1     Running   0             35m
# shippingservice-5459f64756-87mwl         1/1     Running   0             39m
```

The problems with the currencyservice and paymentservice seem to be solved.

**Step 4:** Browse the application\
We execute the following command to get the external IP address of the frontend service:
```sh
kubectl get services -n microservices
# =>
# NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
# adservice               ClusterIP      10.128.251.254   <none>           9555/TCP       39m
# cartservice             ClusterIP      10.128.71.146    <none>           7070/TCP       39m
# checkoutservice         ClusterIP      10.128.26.171    <none>           5050/TCP       35m
# currencyservice         ClusterIP      10.128.151.69    <none>           7000/TCP       37s
# emailservice            ClusterIP      10.128.248.152   <none>           5000/TCP       40m
# frontend                LoadBalancer   10.128.151.91    172.105.146.79   80:31829/TCP   35m
# paymentservice          ClusterIP      10.128.103.97    <none>           50051/TCP      38s
# productcatalogservice   ClusterIP      10.128.33.135    <none>           3550/TCP       39m
# recommendationservice   ClusterIP      10.128.174.59    <none>           8080/TCP       40m
# redis-cart              ClusterIP      10.128.92.165    <none>           6379/TCP       35m
# shippingservice         ClusterIP      10.128.161.92    <none>           50051/TCP      39m
```

We open the browser and navigate to `http://172.105.146.79` to see the microservices application "boutique" in action.