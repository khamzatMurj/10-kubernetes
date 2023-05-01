## Demo Project - Setup Prometheus Monitoring in Kubernetes Cluster

### Topics of the Demo Project
Setup Prometheus monitoring in a Kubernetes Cluster

### Technologies Used
- Kubernetes
- Helm
- Prometheus

### Project Description
- Deploy Prometheus in local Kubernetes cluster using a Helm chart
- Access the Grafana UI
- Access the Prometheus UI

#### Steps to deploy Prometheus in a local Kubernetes cluster using a Helm chart
You can use an operator that manages the combination of all the compontents that make up the Prometheus stack as one unit. The most effitient way of deploying an operator is using a Helm chart solving this task. The Helm chart will do the initial setup of the Prometheus Operator, Prometheus Server and Alertmanager, and the operator will then manage the running Prometheus setup.

Open your browser and navigate to the [Prometheus Community on Github](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack). Here you'll find the commands to install the Helm Chart:

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
# =>
# kube-prometheus-stack has been installed. 

# check the status
kubectl get pods -l "release=prometheus"

# Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

****
Optional: Get some information on the most important components

It's important to know is how to add/adjust alert rules and how to adjust Prometheus configuration.

```sh
kubectl get statefulsets
# =>
# NAME                                                   READY   AGE
# alertmanager-prometheus-kube-prometheus-alertmanager   1/1     33m
# prometheus-prometheus-kube-prometheus-prometheus       1/1     33m

kubectl describe prometheus-prometheus-kube-prometheus-prometheus > prometheus.yaml
less prometheus.yaml
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
****

### Steps to access the Grafana UI
```sh
kubectl get services
# => 
# NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
# ...
# prometheus-grafana   ClusterIP   10.107.198.230   <none>        80/TCP    37m
# ...
```

Grafana is an internal service. In production environment we would usually define an ingress to make it accessible from outside of the cluster. To access it for this demo, we just define a port-forwarding:
```sh
kubectl get pods
# =>
# NAME                                                     READY   STATUS    RESTARTS        AGE
# alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   1 (4m50s ago)   38m
# prometheus-grafana-6984c5759f-cd6zw                      3/3     Running   0               38m
# prometheus-kube-prometheus-operator-655c5b45c7-tlfvc     1/1     Running   0               38m
# prometheus-kube-state-metrics-7fbdd95dc4-bllf5           1/1     Running   0               38m
# prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0               38m
# prometheus-prometheus-node-exporter-jw2gg                1/1     Running   0               38m

kubectl port-forward prometheus-grafana-6984c5759f-cd6zw 3000
```

Now we can access the Grafana UI in the browser on `localhost:3000`, username `admin` and password `prom-operator` (looked up in the prometheus operator [chart documentation](https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/values.yaml) -> `adminPassword`).

### Steps to access Prometheus UI
Prometheus itself provides a UI too. It can be access on port 9090 of the related pod, so let's setup a port-forwarding too:
```sh
kubectl port-forward prometheus-prometheus-kube-prometheus-prometheus-0 9090
```

Now we can access the Prometheus UI on `localhost:9090`. Under 'Status' > 'Rules' we can find all the alerting rules and under 'Status' > 'Targets' we can find all the metric endpoints that are being scraped.
