# Voting Application with Grafana & Prometheus Monitoring

This project demonstrates how to deploy a voting application on Kubernetes with monitoring capabilities using Grafana and Prometheus.

## Prerequisites

- Ubuntu or similar Linux distribution
- Docker installed
- kubectl installed
- Helm installed
- Kind (Kubernetes in Docker) installed

## Installation Steps

### 1. Clone the Repository

```bash
git clone https://github.com/devops-bharat05/Voting-Application-Grafana-Prometheus.git
cd Voting-Application-Grafana-Prometheus
```

### 2. Setup Kubernetes Cluster using Kind

Navigate to the kind-cluster directory and set up the cluster:

```bash
cd kind-cluster
chmod +700 *.sh
./install_kind.sh
kind create cluster --config=config.yml --name=my-cluster
```

### 3. Deploy the Voting Application

Navigate to the k8s-specifications directory and deploy the application:

```bash
cd ../k8s-specifications
kubectl apply -f .
```

Verify the deployment:
```bash
kubectl get pods -A
kubectl get all
```

### 4. Set up Monitoring Stack

1. Add required Helm repositories:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

2. Create monitoring namespace:
```bash
kubectl create namespace monitoring
```

3. Install Prometheus and Grafana using Helm:
```bash
helm install kind-prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.service.nodePort=30000 \
  --set prometheus.service.type=NodePort \
  --set grafana.service.nodePort=31000 \
  --set grafana.service.type=NodePort \
  --set alertmanager.service.nodePort=32000 \
  --set alertmanager.service.type=NodePort \
  --set prometheus-node-exporter.service.nodePort=32001 \
  --set prometheus-node-exporter.service.type=NodePort
```

### 5. Access the Applications

Set up port forwarding to access the services:

```bash
# For Prometheus
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0

# For Grafana
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0

# For Voting Application
kubectl port-forward svc/vote 5000:5000 --address=0.0.0.0
```

### 6. Access Points

- Prometheus: http://localhost:9090
- Grafana: http://localhost:31000
- Voting Application: http://localhost:5000
- Results Application: http://localhost:31001

### 7. Service Details

```bash
NAME                                       TYPE        CLUSTER-IP      PORT(S)
alertmanager-operated                      ClusterIP   None           9093/TCP,9094/TCP,9094/UDP
kind-prometheus-grafana                    NodePort    10.96.252.27   80:31000/TCP
kind-prometheus-kube-prome-alertmanager    NodePort    10.96.215.204  9093:32000/TCP,8080:30118/TCP
kind-prometheus-kube-prome-prometheus      NodePort    10.96.231.63   9090:30000/TCP,8080:30307/TCP
kind-prometheus-prometheus-node-exporter   NodePort    10.96.150.16   9100:32001/TCP
```

## Monitoring Stack Components

- **Prometheus**: For metrics collection and storage
- **Grafana**: For metrics visualization and dashboarding
- **AlertManager**: For handling alerts
- **Node Exporter**: For collecting host metrics
- **Kube State Metrics**: For collecting Kubernetes object metrics

## Default Credentials

- **Grafana**:
  - Username: admin
  - Password: prom-operator

## Troubleshooting

If you encounter port conflicts while setting up port forwarding, ensure that:
1. No other services are using the required ports
2. Previous port-forward processes are terminated
3. You have the necessary permissions to bind to the specified ports

## Application Architecture

The voting application consists of:
- Frontend voting interface
- Redis for temporary storage
- Worker for processing votes
- PostgreSQL for permanent storage
- Results interface for displaying voting results

Each component is deployed as a separate Kubernetes deployment with corresponding services.
