
## Setup Instruction Multi-Tenant Kubernetes Cluster with NGINX Ingress and Route 53 DNS

## Overview

Welcome to the repository for setting up a multi-tenant Kubernetes cluster with NGINX Ingress and AWS Route 53 for subdomain-based routing. This guide demonstrates how to deploy multiple services and route traffic efficiently using subdomains.

## What You’ve Accomplished

- Configured **Route 53** to manage subdomains:
  - `app1.891376932007.realhandsonlabs.net`
  - `app2.891376932007.realhandsonlabs.net`
- Deployed backend services:
  - `app1-service`
  - `app2-service`
    
- Set up an **Ingress resource** to route traffic based on subdomains.
- Verified the setup using **browser** and **curl**.

---

## Prerequisites

1. A Kubernetes cluster (self-hosted or managed, such as EKS, AKS, or GKE).
2. AWS Route 53 hosted zone for your domain.
3. Installed tools:
   - `kubectl`
   - `helm`
   - AWS CLI.
4. Basic understanding of Kubernetes Ingress and DNS.s

### 1. Configure Route 53

- Add A and CNAME records in your hosted zone to point to the cluster’s NGINX Ingress controller.
- Example configuration for subdomains:
  - `app1.891376932007.realhandsonlabs.net`
  - `app2.891376932007.realhandsonlabs.net`

### 2. Deploy NGINX Ingress Controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=LoadBalancer
```

- Confirm the Ingress controller’s external IP:

```bash
kubectl get svc -n ingress-nginx
```

### 3. Deploy Backend Services

Create and apply Kubernetes manifests for `app1-service` and `app2-service`. Example:

#### `app1-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app1
        image: nginx:latest
        ports:
        - containerPort: 80
```

#### `app1-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-service
spec:
  selector:
    app: app1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

Repeat for `app2-service` with appropriate metadata.

### 4. Set Up Ingress Resource

#### `ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-tenant-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - host: app1.891376932007.realhandsonlabs.net
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  - host: app2.891376932007.realhandsonlabs.net
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

Apply the Ingress resource:

```bash
kubectl apply -f ingress.yaml
```

### 5. Verify the Setup

- **Using the browser:** Visit `http://app1.891376932007.realhandsonlabs.net` and `http://app2.891376932007.realhandsonlabs.net`.
- **Using curl:**

```bash
curl http://app1.891376932007.realhandsonlabs.net
curl http://app2.891376932007.realhandsonlabs.net
```

---

## Next Steps

- Scale services dynamically using Horizontal Pod Autoscaler.
- Integrate SSL/TLS certificates using Let’s Encrypt and Cert-Manager.
- Set up monitoring for the cluster using Prometheus and Grafana.
- Optimize the Ingress configuration for better performance and security.

---

## Repository Structure

```
.
├── manifests/
│   ├── app1-deployment.yaml
│   ├── app1-service.yaml
│   ├── app2-deployment.yaml
│   ├── app2-service.yaml
│   └── ingress.yaml
├── README.md
└── setup-scripts/
    ├── deploy-nginx-ingress.sh
    └── verify-setup.sh
```

---

## Contributing

Feel free to submit issues or pull requests to improve this project.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

