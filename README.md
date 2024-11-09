# 🚀 Getting Started with Kubernetes and Service Mesh: A Practical Guide

## 📚 What is Kubernetes?

Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications. Think of it as an intelligent system that helps you:

- 🔄 Automatically distribute and balance application workloads
- 🏥 Self-heal applications when things go wrong
- 📈 Scale applications up or down based on demand
- 🔄 Perform rolling updates with zero downtime

### Key Concepts in Kubernetes:

1. **Pods** 📦: Smallest deployable units that contain one or more containers
2. **Services** 🌐: Network abstractions that expose pods to other components
3. **Deployments** 📋: Manage the lifecycle of pods and ReplicaSets
4. **ConfigMaps/Secrets** 🔐: Store configuration and sensitive data
5. **Namespaces** 📂: Virtual clusters for resource isolation

## 🕸️ What is Service Mesh?

A service mesh is a dedicated infrastructure layer that handles service-to-service communication in microservices architectures. It provides:

- 🔒 Security (mTLS)
- 📊 Monitoring and tracing
- 🔄 Load balancing
- 🏥 Circuit breaking
- 🚦 Traffic management

### Why Do You Need a Service Mesh?

1. **Complexity Management**: As your microservices grow, managing communication becomes challenging
2. **Security**: Automatic encryption of all service-to-service communication
3. **Observability**: Detailed insights into service behavior and performance
4. **Traffic Control**: Advanced routing and deployment strategies

## 💻 Setting Up Your Learning Environment

### Prerequisites:
- GitHub account and GitHub Codespaces
- Basic command-line knowledge

### Step 1: Start a Codespace

1. Go to GitHub and create a new repository
2. Click on "Code" button and select "Open with Codespaces"
3. Click "New codespace"

### Step 2: Install Required Tools

```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Step 3: Start Minikube

```bash
# Start Minikube with Docker driver
minikube start --driver=docker --cpus=2 --memory=4096mb --addons=ingress

# Verify cluster is running
kubectl cluster-info
minikube status
```

### Step 4: Enable Required Addons

```bash
# Enable useful addons
minikube addons enable metrics-server
minikube addons enable dashboard
minikube addons enable ingress

# Verify addons status
minikube addons list
```

## 🔧 Installing Istio Service Mesh

```bash
# Download Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH

# Install Istio with demo profile
istioctl install --set profile=demo -y

# Enable Istio injection for default namespace
kubectl label namespace default istio-injection=enabled

# Verify Istio installation
kubectl get pods -n istio-system
```

## 📊 Deploying Sample Applications

```bash
# Deploy sample bookinfo application
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

# Verify all pods are running
kubectl get pods

# Deploy gateway and virtual service
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

# Get Minikube IP
minikube ip

# Set up ingress
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: bookinfo-ingress
  namespace: default
spec:
  rules:
  - host: bookinfo.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: productpage
            port:
              number: 9080
EOF
```

## 🔍 Setting Up Observability

### Install Monitoring Stack:

```bash
# Install Kiali and other addons
kubectl apply -f samples/addons/kiali.yaml
kubectl apply -f samples/addons/prometheus.yaml
kubectl apply -f samples/addons/grafana.yaml
kubectl apply -f samples/addons/jaeger.yaml

# Wait for deployments
kubectl wait --for=condition=available --timeout=300s deployment/kiali -n istio-system
kubectl wait --for=condition=available --timeout=300s deployment/prometheus -n istio-system
kubectl wait --for=condition=available --timeout=300s deployment/grafana -n istio-system
```

## 🎯 Accessing the Services

```bash
# Start Minikube tunnel in a separate terminal
minikube tunnel

# Access the services (in separate terminals)
kubectl port-forward svc/kiali 20001:20001 -n istio-system
kubectl port-forward svc/grafana 3000:3000 -n istio-system
kubectl port-forward svc/prometheus 9090:9090 -n istio-system
```

### Generate Sample Traffic:

```bash
# Get the ingress IP
export INGRESS_IP=$(minikube ip)

# Add to /etc/hosts (requires sudo)
echo "$INGRESS_IP bookinfo.local" | sudo tee -a /etc/hosts

# Generate traffic
while true; do curl -s http://bookinfo.local/productpage > /dev/null; sleep 0.5; done
```

### Access Dashboard URLs:
- BookInfo: http://bookinfo.local/productpage
- Kiali: http://localhost:20001
- Grafana: http://localhost:3000
- Prometheus: http://localhost:9090

## 📊 Exploring the Service Mesh

### View Service Topology in Kiali:
1. Open Kiali dashboard (http://localhost:20001)
2. Go to Graph section
3. Select the 'default' namespace
4. Watch real-time traffic flow

### Monitor Metrics in Grafana:
1. Open Grafana (http://localhost:3000)
2. Navigate to Dashboards -> Browse
3. Look for Istio Service/Workload/Mesh dashboards

## 🧪 Experimenting with Traffic Management

### Example: Traffic Splitting

```bash
# Create a v2 deployment of reviews service
kubectl apply -f samples/bookinfo/platform/kube/bookinfo-reviews-v2.yaml

# Apply traffic splitting rule
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 50
    - destination:
        host: reviews
        subset: v2
      weight: 50
EOF
```

## 🧹 Cleanup

```bash
# Delete the application
kubectl delete -f samples/bookinfo/platform/kube/bookinfo.yaml

# Remove Istio configurations
kubectl delete -f samples/bookinfo/networking/bookinfo-gateway.yaml

# Stop Minikube
minikube stop

# Delete the cluster
minikube delete
```

## 📝 Next Steps

1. Learn advanced Kubernetes concepts:
   - StatefulSets
   - DaemonSets
   - PersistentVolumes
   - Custom Resources

2. Explore Istio features:
   - Security Policies
   - Rate Limiting
   - Circuit Breaking
   - Fault Injection

3. Practice with real applications:
   - Deploy your own microservices
   - Implement blue-green deployments
   - Set up mutual TLS
   - Create custom routing rules

## 🔧 Troubleshooting Tips

1. **Check Minikube Status**:
```bash
minikube status
```

2. **View Pod Logs**:
```bash
kubectl logs <pod-name> -n <namespace>
```

3. **Describe Resources**:
```bash
kubectl describe pod <pod-name>
kubectl describe service <service-name>
```

4. **Restart Minikube**:
```bash
minikube stop
minikube start
```

## 📚 Additional Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Istio Documentation](https://istio.io/latest/docs/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Kiali Documentation](https://kiali.io/docs/)