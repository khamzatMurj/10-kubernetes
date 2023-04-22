## Demo Project - Deployment with ConfigMap and Secret Volume Types

### Topics of the Demo Project
Deploy Mosquitto message broker with ConfigMap and Secret Volume Types

### Technologies Used
- Kubernetes
- Docker
- Mosquitto

### Project Description
- Define configuration and passwords for Mosquitto message broker with ConfigMap and Secret Volume types

#### Steps to define a ConfigMap holding the content of the mosquitto config file
Create a file called `mosquitto-configmap.yaml` with the following content:
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
```

#### Steps to define a Secret holding a secret file
The secret file should contain the text 'some supersecret file contents nobody should see'. In Secrets we have to provide the content base64 encoded, so execute:
```sh
echo -n 'some supersecret file contents nobody should see' | base64
# => c29tZSBzdXBlcnNlY3JldCBmaWxlIGNvbnRlbnRzIG5vYm9keSBzaG91bGQgc2Vl
```

Now create a file called `mosquitto-secret.yaml` with the following content:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mosquitto-secret-file
type: Opaque
data:
  secret.file: |
    c29tZSBzdXBlcnNlY3JldCBmaWxlIGNvbnRlbnRzIG5vYm9keSBzaG91bGQgc2Vl
```

#### Steps to define a Deployment for a mosquitto Pod
Create a file called `mosquitto-deployment.yaml` with the following content:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mosquitto-depl
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

#### Steps to create the components and check the content of the files
Apply these configuration files:
```sh
kubectl apply -f mosquitto-configmap.yaml
kubectl apply -f mosquitto-secret.yaml
kubectl apply -f mosquitto-deployment.yaml

kubectl get pods
# NAME                         READY   STATUS              RESTARTS   AGE
# mosquitto-65f5cbcbc5-cxhvj   0/1     ContainerCreating   0          4s

kubectl get pod mosquitto-65f5cbcbc5-55x7c --watch
# NAME                         READY   STATUS              RESTARTS   AGE
# mosquitto-65f5cbcbc5-cxhvj   0/1     ContainerCreating   0          7s
# mosquitto-65f5cbcbc5-55x7c   1/1     Running             0          15s
```

Enter the mosquitto container and check the content of the secret file:
```sh
kubectl exec -it mosquitto-65f5cbcbc5-55x7c -- /bin/sh
  cat /mosquitto/secret/secret.file
  # => some supersecret file contents nobody should see
  exit
```
