## 📘 Kubernetes Persistent Storage (PV & PVC) – Hands-on with Nginx

This guide walks through a **complete, working setup** of Kubernetes storage using **Persistent Volume (PV)** and **Persistent Volume Claim (PVC)**, and how to attach it to a **Deployment (Nginx)**.

You can copy-paste and follow step-by-step.

---

# 🚀 What You Will Learn

* What is PV and PVC
* How PV and PVC bind
* How to attach storage to a pod
* How to debug common errors
* Real Kubernetes storage workflow

---

# 🛠️ Prerequisites

```bash
docker --version
kubectl version
kind --version   # or minikube
```

---

# 🧠 Basic Concepts (Very Simple)

* **PV (Persistent Volume)** → actual storage (disk inside node)
* **PVC (Persistent Volume Claim)** → request for that storage
* **Pod/Deployment** → uses PVC to access storage

👉 Flow:

```
PV → PVC → Pod → Container
```

---

# 📁 Project Structure

```
Persistant_Volume/
│── pv.yml
│── pvc.yml
│── deployment.yml
```

---

# 💾 Step 1: Create Persistent Volume

```yaml
# pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
  labels:
    app: locals
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
```

Apply:

```bash
kubectl apply -f pv.yml
```

---

# 📂 Step 2: Create Persistent Volume Claim

```yaml
# pvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
  namespace: nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
```

Apply:

```bash
kubectl apply -f pvc.yml
```

---

# 🔍 Verify Binding

```bash
kubectl get pv
kubectl get pvc -n nginx
```

Expected:

```
STATUS: Bound
```

---

# 🚀 Step 3: Create Deployment with Storage

```yaml
# deployment.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80

        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi

        volumeMounts:
        - name: my-vol
          mountPath: /usr/share/nginx/html   # important

      volumes:
      - name: my-vol
        persistentVolumeClaim:
          claimName: local-pvc
```

Apply:

```bash
kubectl apply -f deployment.yml
```

---

# 🔍 Step 4: Check Everything

```bash
kubectl get pods -n nginx
kubectl get pvc -n nginx
kubectl get pv
```

---

# 📦 Verify Storage (Optional but Important)

Enter node container:

```bash
docker exec -it <kind-node-name> bash
cd /mnt/data
```

👉 This is your actual persistent storage

---

# ⚠️ Common Errors (Very Important)

## ❌ Pod stuck in Pending

**Reason:**

* PVC not bound

✔ Fix:

```bash
kubectl get pvc -n nginx
```

---

## ❌ CreateContainerConfigError

**Reason:**

* Wrong PVC name or missing config

✔ Fix:

```bash
kubectl describe pod <pod-name> -n nginx
```

---

## ❌ Storage not working

**Reason:**

* Wrong mount path

✔ Fix:

```yaml
mountPath: /usr/share/nginx/html
```

---

## ❌ Namespace Issue

* PVC and Deployment must be in same namespace

---

# 🔄 How It Works (Clear Flow)

1. PV created → storage ready
2. PVC created → requests storage
3. PV + PVC → bind
4. Deployment created → uses PVC
5. Pod mounts storage → data persists

---

# 🧠 Key Learnings

* PV is cluster-level storage
* PVC is namespace-level request
* Pod never uses PV directly
* Storage remains even after pod restart
* Kubernetes manages binding automatically

---

# 📌 Final Result

* Nginx deployed successfully
* Persistent storage attached
* Data stored in `/mnt/data`
* Pods can restart without losing data

---

# 🔥 Next Steps

* StatefulSet with storage (MySQL)
* StorageClass (dynamic provisioning)
* Real database deployment

---

## ⭐ If this helped you, give this repo a star

