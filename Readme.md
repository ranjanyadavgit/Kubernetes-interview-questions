# 🚀 Kubernetes CNI Fundamentals – Hands-on PoC

## 📌 Overview

This Proof of Concept (PoC) demonstrates Kubernetes networking fundamentals with a focus on the **Container Network Interface (CNI)** specification and popular plugins such as **Flannel, Calico, and Cilium**.

The lab explores key Kubernetes networking challenges including:

* Pod-to-Pod communication
* Service discovery
* Traffic routing
* Network security enforcement

It also compares **overlay and underlay networking models** and highlights how different CNI plugins implement networking efficiently.

---

## 🎯 Objectives

By completing this PoC, you will:

* Understand Kubernetes networking limitations and reliance on CNI
* Deploy applications to validate Pod networking
* Install and compare Flannel, Calico, and Cilium
* Implement NetworkPolicies
* Observe real-time traffic using eBPF (Cilium Hubble)
* Troubleshoot Kubernetes networking issues

---

## 🏗️ Architecture

```
                 +---------------------------+
                 | Kubernetes Cluster        |
                 |                           |
                 |   CNI Plugin              |
                 | (Flannel / Calico / Cilium)
                 +------------+--------------+
                              |
         ----------------------------------------------
         |                     |                      |
      Frontend Pod         Backend Pod            DB Pod
         |                     |
         |-----> Service (ClusterIP) --------------|
```

---

## ⚙️ Prerequisites

* Docker installed
* kubectl installed
* Kind / Minikube / Cloud Kubernetes cluster
* Helm (for Cilium)
* Basic Kubernetes knowledge

---

## 🚀 Step 1 – Create Cluster (Kind)

```bash
kind create cluster --name cni-poc
kubectl cluster-info
```

Verify system pods:

```bash
kubectl get pods -n kube-system
```

---

## 🧪 Step 2 – Deploy Sample Applications

### Frontend Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

### Backend Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: httpd
        image: httpd
```

Apply resources:

```bash
kubectl apply -f frontend.yaml
kubectl apply -f backend.yaml
```

---

## 🔍 Step 3 – Validate Pod Networking

```bash
kubectl get pods -o wide
kubectl exec -it <frontend-pod> -- curl <backend-pod-ip>
```

✔️ Confirms Pod-to-Pod communication

---

## 🌐 Step 4 – Service Discovery

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  selector:
    app: backend
  ports:
    - port: 80
```

```bash
kubectl apply -f backend-svc.yaml
kubectl exec -it <frontend-pod> -- curl backend-svc
```

✔️ Confirms DNS resolution and kube-proxy routing

---

## 🔁 Step 5 – Install Flannel (Overlay Networking)

```bash
kubectl delete daemonset kindnet -n kube-system
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

✔️ Uses VXLAN overlay networking

---

## 🔐 Step 6 – Install Calico (Network Policies)

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Sample Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

✔️ Restricts backend access to frontend pods only

---

## ⚡ Step 7 – Install Cilium (eBPF Observability)

```bash
helm repo add cilium https://helm.cilium.io
helm install cilium cilium/cilium --namespace kube-system
```

Enable Hubble:

```bash
cilium hubble enable
cilium hubble ui
```

✔️ Provides real-time network visibility

---

## 🧠 Overlay vs Underlay Comparison

| Feature     | Overlay (VXLAN) | Underlay (BGP)   |
| ----------- | --------------- | ---------------- |
| Example     | Flannel         | Calico BGP       |
| Performance | Moderate        | High             |
| Complexity  | Low             | Medium           |
| Routing     | Encapsulation   | Native routing   |
| Use case    | Small clusters  | Production scale |

---

## 🧪 Troubleshooting Commands

```bash
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl get svc
kubectl get networkpolicy
kubectl logs -n kube-system <cni-pod>
```

---

## 📊 Key Learnings

* Kubernetes delegates networking to CNI plugins
* Flannel provides simple overlay networking
* Calico offers policy-based security and BGP routing
* Cilium leverages eBPF for observability and performance
* NetworkPolicies are critical for zero-trust Kubernetes security

---

## ⭐ Future Enhancements

* Deploy PoC on EKS / AKS / GKE
* Integrate Istio service mesh
* Add MetalLB for on-prem load balancing
* Implement multi-cluster networking
* Capture traffic with tcpdump inside pods

---

## 👨‍💻 Author

**Ranjan Yadav**
DevOps Engineer | Kubernetes | Cloud | DevSecOps

---

## 📜 License

This project is for learning and demonstration purposes.
