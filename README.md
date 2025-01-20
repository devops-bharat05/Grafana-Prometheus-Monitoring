# Voting Application with Grafana & Prometheus Monitoring

This project demonstrates how to deploy a voting application on Kubernetes with monitoring capabilities using Grafana and Prometheus.

## Prerequisites

- Ubuntu or similar Linux distribution
- Docker installed
- kubectl installed
- Helm installed
- Kind (Kubernetes in Docker) installed

## Detailed Installation Steps

### 1. Setting Up Kubernetes Cluster with Kind

Create a 3-node Kubernetes cluster:
```bash
kind create cluster --config=config.yml
```

Verify cluster setup:
```bash
kubectl cluster-info --context kind-kind
kubectl get nodes
kind get clusters
```

### 2. Installing kubectl

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### 3. Deploy the Voting Application

Clone the repository:
```bash
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app/
```

Deploy the application:
```bash
kubectl apply -f k8s-specifications/
```

Verify deployment:
```bash
kubectl get all
```

Set up port forwarding:
```bash
kubectl port-forward service/vote 5000:5000 --address=0.0.0.0 &
kubectl port-forward service/result 5001:5001 --address=0.0.0.0 &
```

### 4. Install Kubernetes Dashboard (Optional)

Deploy the dashboard:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

Create admin token:
```bash
kubectl -n kubernetes-dashboard create token admin-user
```

### 5. Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### 6. Set up Monitoring Stack

Add Helm repositories:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

Create monitoring namespace:
```bash
kubectl create namespace monitoring
```

Install Prometheus stack:
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

Verify installation:
```bash
kubectl get svc -n monitoring
kubectl get namespace
```

### 7. Access Monitoring Tools

Set up port forwarding:
```bash
kubectl port-forward svc/kind-prometheus-kube-prome-prometheus -n monitoring 9090:9090 --address=0.0.0.0 &
kubectl port-forward svc/kind-prometheus-grafana -n monitoring 31000:80 --address=0.0.0.0 &
```

## Prometheus Queries for Monitoring

### CPU Usage
Monitor CPU usage percentage across the default namespace:
```
sum (rate (container_cpu_usage_seconds_total{namespace="default"}[1m])) / sum (machine_cpu_cores) * 100
```

![Alt text](/img/Screenshot 2025-01-20 191256.png)

### Memory Usage
Track memory usage by pod in the default namespace:
```
sum (container_memory_usage_bytes{namespace="default"}) by (pod)
```

### Network Traffic
Monitor network receive traffic (incoming):
```
sum(rate(container_network_receive_bytes_total{namespace="default"}[5m])) by (pod)
```

Monitor network transmit traffic (outgoing):
```
sum(rate(container_network_transmit_bytes_total{namespace="default"}[5m])) by (pod)
```

## Access Points

- Prometheus: http://localhost:9090
- Grafana: http://localhost:31000
- Voting Application: http://localhost:5000
- Results Application: http://localhost:5001

## Default Credentials

- **Grafana**:
  - Username: admin
  - Password: prom-operator

## Cleanup

To delete the Kind cluster:
```bash
kind delete cluster --name=kind
```

## Troubleshooting

If you encounter port conflicts while setting up port forwarding, ensure that:
1. No other services are using the required ports
2. Previous port-forward processes are terminated
3. You have the necessary permissions to bind to the specified ports

## Monitoring Stack Components

- **Prometheus**: For metrics collection and storage
- **Grafana**: For metrics visualization and dashboarding
- **AlertManager**: For handling alerts
- **Node Exporter**: For collecting host metrics
- **Kube State Metrics**: For collecting Kubernetes object metrics

## Application Architecture

The voting application consists of:
- Frontend voting interface
- Redis for temporary storage
- Worker for processing votes
- PostgreSQL for permanent storage
- Results interface for displaying voting results

Each component is deployed as a separate Kubernetes deployment with corresponding services.
