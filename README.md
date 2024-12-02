```markdown
# kubeSandbox

This repository contains instructions and YAML files to create a Kubernetes cluster using Minikube, along with namespaces, nodes, pods, services, and NodePorts.

## Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## Table of Contents

1. [Start Minikube Cluster](#1-start-minikube-cluster)
2. [Create Namespace](#2-create-namespace)
3. [Create Nodes](#3-create-nodes)
4. [Deploy Pods](#4-deploy-pods)
5. [Expose Pods with Services](#5-expose-pods-with-services)
6. [Access Services using NodePort](#6-access-services-using-nodeport)

## 1. Start Minikube Cluster

Start a Minikube cluster with multiple nodes:

```bash
minikube start --nodes 3 --cpus 2 --memory 2048
```

Verify the status:

```bash
minikube status
kubectl get nodes -o wide
```

## 2. Create Namespace

Create a new namespace named `kernelns`:

```bash
kubectl create namespace kernelns --dry-run=client --output yaml > namespace.yaml
kubectl apply -f namespace.yaml
```

## 3. Create Nodes

Minikube nodes are created as part of the cluster initialization. Verify nodes:

```bash
kubectl get nodes -o wide
```

## 4. Deploy Pods

Create YAML files to deploy pods.

### client-app Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: client-app
  namespace: kernelns
  labels:
    app: client-app
spec:
  containers:
  - name: client-app-container
    image: nginx:latest
    ports:
    - containerPort: 80
  nodeSelector:
    kubernetes.io/hostname: minikube-m02
```

### backend-pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend-pod
  namespace: kernelns
  labels:
    app: backend-pod
spec:
  containers:
  - name: backend-container
    image: nginx:latest
    ports:
    - containerPort: 80
  nodeSelector:
    kubernetes.io/hostname: minikube-m02
```

Apply the YAML files:

```bash
kubectl apply -f client-app-pod.yaml
kubectl apply -f backend-pod.yaml
```

Verify the pods:

```bash
kubectl get pods -n kernelns -o wide
```

## 5. Expose Pods with Services

Create YAML files for services.

### client-app Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: client-app-service
  namespace: kernelns
spec:
  type: NodePort
  selector:
    app: client-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30001
```

### backend-pod Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-pod-service
  namespace: kernelns
spec:
  type: NodePort
  selector:
    app: backend-pod
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30002
```

Apply the YAML files:

```bash
kubectl apply -f client-app-service.yaml
kubectl apply -f backend-pod-service.yaml
```

Verify the services:

```bash
kubectl get services -n kernelns
```

## 6. Access Services using NodePort

Find the Minikube IP address:

```bash
minikube ip
```

Access the services using the node IP and NodePort.

### Access `client-app-service`

```text
http://<minikube-ip>:30001
```

### Access `backend-pod-service`

```text
http://<minikube-ip>:30002
```

Ensure that the Minikube tunnel is running if necessary:

```bash
minikube tunnel
