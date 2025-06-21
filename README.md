# 🔗 DevOps Project 03: Kubernetes Deployment with Helm Charts + Monitoring

This guide documents the steps followed to create a **DevOps project** focused on deploying a Flask or Nginx application into **Kubernetes** using **Minikube**, along with monitoring setup via **Prometheus + Grafana**. The application was also packaged using **Helm charts** to simplify deployment and configuration.

---

## 🔗 Objectives Completed

* Installed and used **Minikube** and optionally K3d
* Deployed a Flask or Nginx app using Kubernetes YAML and Helm
* Set up monitoring with **Prometheus + Grafana**
* Packaged the app into a reusable **Helm chart**

---

## 🔗 Project Structure Overview

```
devops_project_03_helm/
├── flask-app/
│   ├── app.py
│   ├── Dockerfile
│   └── requirements.txt
├── charts/
│   └── flask-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
├── prometheus-grafana/
│   ├── prometheus.yaml
│   ├── grafana.yaml
├── README.md
```

## 🔗 Project Code Files (Flask App, Helm Chart, Monitoring YAMLs)

### `flask-app/app.py`

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return "Hello from Flask inside Kubernetes!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

---

### `flask-app/Dockerfile`

```Dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

CMD ["python", "app.py"]
```

---

### `flask-app/requirements.txt`

```
flask
```

---

### `charts/flask-app/Chart.yaml`

```yaml
apiVersion: v2
name: flask-app
description: A simple Flask app deployed via Helm
type: application
version: 0.1.0
appVersion: "1.0"
```

---

### `charts/flask-app/values.yaml`

```yaml
replicaCount: 1

image:
  repository: your-dockerhub-username/flask-app
  pullPolicy: IfNotPresent
  tag: latest

service:
  type: NodePort
  port: 5000

resources: {}

nodeSelector: {}
tolerations: []
affinity: {}
```

---

### `charts/flask-app/templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "flask-app.fullname" . }}
  labels:
    {{- include "flask-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
        - name: flask
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 5000
```

---

### `charts/flask-app/templates/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: {{ .Values.service.type }}
  selector:
    app: flask
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 5000
```

---

### `prometheus-grafana/prometheus.yaml`

```yaml
alertmanager:
  enabled: false

pushgateway:
  enabled: false

server:
  service:
    type: NodePort
    nodePort: 30090
```

---

### `prometheus-grafana/grafana.yaml`

```yaml
adminPassword: admin
service:
  type: NodePort
  nodePort: 30030
```


## 🔗 Deployment Architecture Followed

<p align="center">
  <img src="kube_helm.png" alt="Kubernetes with Helm Flowchart" width="350"/>
</p>

---

## 🔗 System Prerequisites

* Ubuntu 20.04 or later
* Docker, kubectl, and Helm installed

### 🔗 Commands to Verify Setup

Check if Minikube is installed:

```bash
which minikube
```

If not present, Minikube was installed using:

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
```

Verify:

```bash
minikube version
```

Install kubectl (if not already available):

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubectl
```

### 🔗 Check Cluster Status and Drivers

The following commands helped verify the current state and configuration of Minikube:

```bash
minikube status        # displays the status of the Minikube cluster (running, paused, stopped)
minikube profile list  # lists all Minikube profiles available on the machine
```
**minikube drivers** shows where Minikube can run the Kubernetes cluster whether inside Docker, a VM, or bare metal, depending on the installed and compatible drivers.
> `minikube drivers` was previously used in older versions to list supported container drivers, but the command has since been deprecated or removed in the latest versions. Refer to the Minikube documentation for updated ways to check available drivers.

### 🔗 Confirm Tool Versions

These commands were used to verify that the installed versions of tools are available and functioning as expected:

```bash
minikube version          # shows the version of Minikube installed
helm version              # displays Helm version and client details
kubectl version --client  # outputs the client version of kubectl installed
```
---

## 🔗 Step 1: Start Minikube (Kubernetes Cluster Creation)

```bash
minikube start --driver=docker
```
This command initializes the Kubernetes control plane and provisions a single-node cluster using Docker as the driver.

> This command can be run from any directory ( home or project folder). However, running it inside the project directory can help keep context and workflows organized.

Optionally, ingress addon was enabled:

```bash
minikube addons enable ingress
```
This command enables the NGINX Ingress Controller inside the Minikube cluster.

##### 🔗 Why it's important:

Allows exposing multiple Kubernetes services under a single external IP or domain.

Enables clean routing rules via URLs (/api, /dashboard).

Supports TLS termination for HTTPS routes.

Essential when managing access to services like Prometheus, Grafana or Flask apps from outside the cluster.

---

### 🔗 Helm Installation Steps (Snap Method)

When trying to install Helm using Snap, the following message appeared:

```bash
sudo snap install helm

```

To proceed, Helm was successfully installed using:

```bash
sudo snap install helm --classic
```

Output:

```
helm 3.18.3 from Snapcrafters✪ installed
```

Verification:

```bash
helm version
```

## 🔗 Common kubectl Commands Used

* `kubectl get nodes` — Displays active nodes
* `kubectl get pods` — Lists all running pods
* `kubectl get services` — Displays service mappings
* `kubectl describe pod <pod-name>` — Shows pod details
* `kubectl logs <pod-name>` — Retrieves container logs
* `kubectl apply -f file.yaml` — Applies Kubernetes configuration
* `kubectl delete -f file.yaml` — Removes resources defined in YAML

---

### 🔗What are `helm create` and `helm install`

#### 🔗 `helm create <chart-name>` 

This command **generates a boilerplate Helm chart directory structure** with default files and templates. It is used to scaffold a new chart that can later be customized and deployed.

**Example:**

```bash
helm create charts/flask-app
```

This creates a directory structure like:

```
charts/flask-app/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    └── _helpers.tpl
```

The files can then be modified (Docker image name, ports, replicas) to fit the application.

#### 🔗 `helm install <release-name> <chart-path>` 

This command **installs (deploys) a Helm chart** to a Kubernetes cluster.

**Example:**

```bash
helm install flask charts/flask-app
```

This installs the `flask-app` Helm chart from the given path with the release name `flask` into the current Kubernetes context.

**Key difference:**

* `helm create` is for chart creation (scaffolding templates).
* `helm install` is for deploying a chart to a live cluster.

Together, these commands form the build → deploy cycle using Helm.

## 🔗 Step 2: Helm Chart Packaging for Flask App

A Helm chart was created for the app:

```bash
helm create charts/flask-app
```

The default templates (`deployment.yaml`, `service.yaml`) were customized to suit the Flask application.

The `values.yaml` file was modified with actual Docker Hub repository info:

```yaml
image:
  repository: <dockerhub-username>/flask-app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 5000
```

---

## 🔗 Step 3: Build & Deploy Flask App with Helm

Docker image was built and pushed:

```bash
cd flask-app
docker build -t <dockerhub-username>/flask-app .
docker push <dockerhub-username>/flask-app
```

Helm was used to deploy the application:

```bash
helm install flask charts/flask-app
```

To confirm deployment:

```bash
kubectl get all
```

To apply changes:

```bash
helm upgrade flask charts/flask-app
```

To remove the deployment:

```bash
helm uninstall flask
```

---

## 🔗 Step 4: Monitoring with Prometheus + Grafana

Helm repos were added:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

Prometheus and Grafana were installed:

```bash
helm install prometheus prometheus-community/prometheus
helm install grafana grafana/grafana --set adminPassword='admin' --set service.type=NodePort
```

Credentials were retrieved:

```bash
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Grafana UI was accessed:

```bash
minikube service grafana
```

Prometheus was added as a data source in Grafana to enable dashboard creation.

---

## 🔗 Outcome

The following milestones were completed:

* A Kubernetes cluster was set up locally using Minikube
* A Flask app was containerized and packaged using Helm
* The app was deployed on Kubernetes via Helm
* Monitoring was added through Prometheus and Grafana
