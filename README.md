

# 🔐 Secure Image Upload Platform on Kubernetes

A **cloud-native, microservices-based image upload platform** built with **Go** and deployed on **Kubernetes**, with a strong emphasis on **security, least-privilege RBAC, ServiceAccount hardening, and network isolation**.

This project demonstrates how to design and deploy a modern microservices application **without granting unnecessary Kubernetes API access to application workloads**.

---

## 🏗️ Architecture Overview

The application follows a **microservices architecture** with a single external entry point.

```
┌─────────────────┐
│   Frontend      │
│   (HTML / JS)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   API Gateway   │  Port: 8081
└────────┬────────┘
         │
 ┌───────┴─────────┐
 │                 │
 ▼                 ▼
┌────────────┐  ┌────────────┐
│ Auth       │  │ Upload     │
│ Service    │  │ Service    │
│ Port 8082  │  │ Port 8083  │
└────────────┘  └────────────┘
```

---

## 🔧 Services

| Service            | Description                                    |
| ------------------ | ---------------------------------------------- |
| **API Gateway**    | Routes requests, serves frontend, handles CORS |
| **Auth Service**   | User registration, login, JWT issuance         |
| **Upload Service** | Secure image uploads and static file serving   |
| **Shared Module**  | Common utilities and JWT validation            |

---

## 🔐 Security Design (Key Focus)

This project intentionally applies **defense-in-depth Kubernetes security controls**:

### ✔ ServiceAccount Hardening

* Dedicated ServiceAccount per microservice
* `automountServiceAccountToken: false`
* No application pod has Kubernetes API credentials

### ✔ Least Privilege RBAC

* **No Roles or RoleBindings created**
* Workloads do not access the Kubernetes API
* If no API access is required, **RBAC = none**

### ✔ Default ServiceAccount Locked Down

* Prevents accidental privilege inheritance
* Forces explicit identity assignment

### ✔ Secrets Management

* JWT secret stored in **Kubernetes Secret**
* Configuration stored in **ConfigMap**
* Designed to be easily replaced by Vault / AWS Secrets Manager

### ✔ Network Policies (Zero Trust)

* Backend services accept traffic **only** from API Gateway
* No lateral pod-to-pod communication
* Explicit ingress and egress rules

### ✔ Pod Security Standards (PSS)

* Namespace enforces `restricted` policy
* Prevents privileged containers
* Blocks privilege escalation

---

## 📂 Repository Structure

```
k8s/
├── 00-pss.yaml
├── 01-common.yaml
├── 02-secrets.yaml
├── 03-auth.yaml
├── 04-serviceaccounts.yaml #sperate the SA account see repo for help
├── 05-api.yaml
├── 06-upload-services.yaml
├── 07-network-policies.yaml
```

---

## 🚀 Deployment Guide

### 1️⃣ Prerequisites

* Go 1.21+
* Docker
* kubectl
* Kubernetes cluster (Minikube, kind, or cloud)

Verify:

```bash
kubectl version --client
docker --version
```

---

### 2️⃣ Create / Start Kubernetes Cluster

Example (Minikube):

```bash
minikube start
```

---

### 3️⃣ Deploy Kubernetes Manifests (Order Matters)

```bash
kubectl apply -f k8s/00-namespace-pss.yaml
kubectl apply -f k8s/01-config.yaml
kubectl apply -f k8s/02-secrets.yaml
kubectl apply -f k8s/03-storage.yaml
kubectl apply -f k8s/04-serviceaccounts.yaml
kubectl apply -f k8s/05-deployments.yaml
kubectl apply -f k8s/06-services.yaml
kubectl apply -f k8s/07-network-policies.yaml
```

---

### 4️⃣ Verify Deployment

```bash
kubectl get pods
kubectl get services
```

View logs:

```bash
kubectl logs -f deployment/api-gateway
```

---

### 5️⃣ Access the Application (Local)

```bash
kubectl port-forward service/api-gateway 8081:8081
```

Open:

```
http://localhost:8081
```

---

## 🔍 Security Validation (Important)

### Verify ServiceAccount isolation

```bash
kubectl describe pod <pod-name> | grep ServiceAccount
```

### Verify Kubernetes API access is blocked

```bash
kubectl exec -it <pod-name> -- sh
curl https://kubernetes.default.svc
```

Expected result:

```
Unauthorized
```

✔ Confirms no API credentials
✔ Confirms least-privilege enforcement

---

## 📊 Autoscaling & Observability

* Horizontal Pod Autoscaler (HPA) enabled
* Kubernetes-native health checks
* Resource requests and limits defined

```bash
kubectl get hpa
kubectl top pods
```

---

## 🧠 Design Philosophy

> **Running in Kubernetes does not mean applications should talk to Kubernetes.**

This project deliberately separates:

* **Cluster orchestration** → kubelet & control plane
* **Application logic** → microservices with no API access

This dramatically reduces the attack surface.

---

## 🎯 Learning Outcomes

* Secure Kubernetes workload identity
* ServiceAccount and RBAC fundamentals
* NetworkPolicy-based zero trust
* Pod Security Standards enforcement
* Production-ready manifest organization

---

## 🔮 Future Enhancements

* External secrets integration (Vault / AWS Secrets Manager)
* Ingress TLS with cert-manager
* OpenTelemetry tracing
* OPA/Gatekeeper policy enforcement

---

## 📄 License

MIT License

---

## ⭐ Final Note

This project intentionally prioritizes **security correctness over convenience** and demonstrates how to deploy microservices in Kubernetes **without unnecessary privileges**.


