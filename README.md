
## Project Overview
This project sets up MongoDB and Mongo Express in a Kubernetes environment, providing a web interface to interact with MongoDB. Additionally, Prometheus and Grafana are integrated to monitor MongoDB and its associated services.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Kubernetes Resources](#kubernetes-resources)
- [Step-by-Step Deployment](#step-by-step-deployment)
- [Setting up Monitoring with Prometheus and Grafana](#setting-up-monitoring-with-prometheus-and-grafana)
- [Accessing Services](#accessing-services)
- [Troubleshooting](#troubleshooting)

## Prerequisites
Make sure you have the following installed:
- Kubernetes Cluster (Minikube or any other)
- `kubectl` CLI
- Helm (v3+)
- NGINX Ingress Controller (if using ingress)
- Docker

Ensure you have an active Kubernetes cluster running before proceeding.

## Kubernetes Resources
This project contains the following key Kubernetes manifest files:
1. **configmap.yaml**: ConfigMap for MongoDB service URL.
2. **secrets.yaml**: Secret for MongoDB credentials (base64 encoded).
3. **mongodb-deployment.yaml**: Deployment for MongoDB.
4. **mongodb-service.yaml**: Service for MongoDB.
5. **mongo_express-deployment.yaml**: Deployment for Mongo Express.
6. **mongo_express-service.yaml**: Service for Mongo Express.
7. **Ingress.yaml**: Ingress resource for accessing Mongo Express through a hostname.
8. **mongodb-pv.yaml**: PersistentVolume and PersistentVolumeClaim for MongoDB storage.


## Step-by-Step Deployment
### Initializing your minikube cluster

```bash
minikube start --cpus 4 --memory 8192
```

### 1. Deploy MongoDB and Mongo Express
Start by applying the Kubernetes manifests for MongoDB and Mongo Express.

```bash
kubectl apply -f configmap.yaml
kubectl apply -f secrets.yaml
kubectl apply -f mongodb-deployment.yaml
kubectl apply -f mongodb-service.yaml
kubectl apply -f mongo_express-deployment.yaml
kubectl apply -f mongo_express-service.yaml
kubectl apply -f Ingress.yaml
kubectl apply -f mongodb-pv.yaml
```


##################################################################################################################################

# Mongodb-and-express-monitoring-
MongoDb-and-MongoExpress deployment using Kubernetes with PV and PVC and adding Prometheus and Grafana setup in Minikube for monitor our Kubernetes Cluster

### Ensure that your Kubernetes cluster has an ingress controller installed to handle Ingress resources
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```


## setting-up-monitoring-with-prometheus-and-grafana
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm repo list
kubectl create namespace monitoring
helm install prometheus prometheus-community/prometheus --namespace monitoring
helm install grafana grafana/grafana --namespace monitoring
```

### we can check the status by running:
 ```bash
kubectl get pods -l app.kubernetes.io/instance=prometheus
```

### To expose the prometheus service
```bash
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-np
```

### This will open a browser window with the Prometheus web interface.
```bash
minikube service prometheus-server-np
```

### To expose the Grafana service
```bash
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-np
```

### This will open a browser window with the Grafana web interface.
```bash
minikube service grafana-np
```

### To get the Grafana admin password, run the following:
```bash
echo "The Grafana uusername is admin & admin password is:" && kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```
##################################################################################################################################

### Configure Prometheus Datasource: 

Once we're logged in to the admin interface, it's time to configure the Prometheus Datasource.
We need to head to Connections > Datasources and add a new Prometheus instance.
Note: The URL for our Prometheus instance is the name of the service http://prometheus-server:80.
![image](https://github.com/user-attachments/assets/1f36582b-d9e8-4fa2-a6e4-9473987c7536)# MongoDB, Mongo Express & Monitoring with Prometheus and Grafana on Kubernetes

###  Configure Grafana Dashboards: 
we'll set up one of the many already available community provided Kubernetes Dashboards.
1- We head to Create (+) > Import section to Import via grafana.com and we set 6417 into the id field and click Load.
2- In the dashboard configuration we need to select the Prometheus Datasource we created in the earlier step.


### 1. install Mongodb-exporter
```bash
helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f values.yaml
```
### 2. Expose the Mongodb-exporter

```bash
kubectl port-forward service/mongodb-exporter-prometheus-mongodb-exporter 9216  
```
## Accessing Services
1. Access Mongo Express
   If you are using the Ingress resource, ensure that you have an Ingress controller (e.g., NGINX) set up in your cluster. Access Mongo Express using the host defined in the Ingress.yaml.
```bash
   http://mongodb.ui.com
```
Ensure the host is added to your local /etc/hosts file for local resolution:

```bash
127.0.0.1 mongodb.ui.com
```
## Troubleshooting
1. Mongo Express Not Accessible: Ensure that Ingress is set up properly and that the host is added to your local DNS resolution (e.g., /etc/hosts).
2. MongoDB Connection Issues: Double-check the Secret and ConfigMap for the correct credentials and service URL.
3. Grafana Not Loading: Ensure the Prometheus and Grafana services are running, and port forwarding is done correctly.

## Flow Chart Diagram

          (Start)
            |
            v
    (Kubernetes Cluster Initialization)
            |
            v
    (MongoDB Deployment)
            |
            v
    (Mongo Express Deployment)
            |
            v
     (MongoDB Exporter Deployment)
            |
            v
    (Set Up Prometheus & Grafana)
            |
            v
    (Access and Monitor Services)
            |
            v
          (End)
