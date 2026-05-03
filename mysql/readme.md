## 📘 Kubernetes MySQL (StatefulSet + Storage) 

This guide helps you **deploy MySQL on Kubernetes using StatefulSet**, with:

* Persistent storage (auto PVC creation)
* ConfigMap (database config)
* Secret (secure password)
* Headless Service (stable networking)

👉 Anyone can follow this and run a **production-like MySQL setup locally**

---

# 🚀 What You Will Achieve

* MySQL cluster with **3 pods**
* Each pod has **its own persistent storage**
* Database automatically created (`devops`)
* Verified inside pod using MySQL CLI

---

# 🧠 Concepts (Simple)

| Component        | Purpose                                 |
| ---------------- | --------------------------------------- |
| StatefulSet      | Manages database pods (stable identity) |
| PVC              | Storage for each pod                    |
| ConfigMap        | Database name                           |
| Secret           | Password                                |
| Headless Service | Pod-to-pod communication                |

---

# 📁 Project Structure

```bash
mysql/
│── namespace.yml
│── configmap.yml
│── secret.yml
│── service.yml
│── statefulset.yml
```

---

# 🛠️ Prerequisites

```bash
docker --version
kubectl version
kind --version   # or minikube
```

---

# 🔹 Step 1: Create Namespace

```yaml
# namespace.yml
apiVersion: v1
kind: Namespace
metadata:
  name: mysql
```

```bash
kubectl apply -f namespace.yml
```

---

# 🔹 Step 2: Create ConfigMap

```yaml
# configmap.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config-map
  namespace: mysql
data:
  MYSQL_DATABASE: devops
```

```bash
kubectl apply -f configmap.yml
```

---

# 🔹 Step 3: Create Secret

```yaml
# secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: mysql
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: cm9vdAo=
```

```bash
kubectl apply -f secret.yml
```

📌 Decode check:

```bash
echo cm9vdAo= | base64 --decode
```

---

# 🔹 Step 4: Create Headless Service

```yaml
# service.yml
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - name: mysql
      port: 3306
      targetPort: 3306
```

```bash
kubectl apply -f service.yml
```

📌 Important:

* `clusterIP: None` → required for StatefulSet

---

# 🔹 Step 5: Create StatefulSet

```yaml
# statefulset.yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
  namespace: mysql

spec:
  serviceName: mysql-service
  replicas: 3

  selector:
    matchLabels:
      app: mysql

  template:
    metadata:
      labels:
        app: mysql

    spec:
      containers:
      - name: mysql
        image: mysql:8.0

        ports:
        - containerPort: 3306

        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD

        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: mysql-config-map
              key: MYSQL_DATABASE

        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql

  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```

```bash
kubectl apply -f statefulset.yml
```

---

# 🔍 Step 6: Verify Pods

```bash
kubectl get pods -n mysql
```

Expected:

```bash
mysql-statefulset-0   Running
mysql-statefulset-1   Running
mysql-statefulset-2   Running
```

---

# 📦 Step 7: Verify Storage (Auto PVC)

```bash
kubectl get pvc -n mysql
```

👉 Output:

```bash
mysql-data-mysql-statefulset-0
mysql-data-mysql-statefulset-1
mysql-data-mysql-statefulset-2
```

---

# 🔐 Step 8: Verify MySQL Inside Pod

```bash
kubectl exec -it mysql-statefulset-0 -n mysql -- bash
```

Then:

```bash
mysql -u root -p
```

Enter password:

```bash
root
```

Run:

```sql
show databases;
```

Expected:

```bash
devops
information_schema
mysql
performance_schema
sys
```

---

# 🔄 StatefulSet Behavior (What You Observed)

You ran:

```bash
kubectl delete pod mysql-statefulset-0 -n mysql
```

👉 Result:

* Pod recreated automatically ✅
* Data still exists ✅

👉 This proves:

* Persistent storage works
* StatefulSet ensures availability

---

# 🔧 Common Errors & Fixes

---

## ❌ CreateContainerConfigError

### Cause:

Wrong key name

```yaml
MYSQSL_DATABASE ❌
```

### Fix:

```yaml
MYSQL_DATABASE ✅
```

---

## ❌ YAML Error (BadIndent / unmarshal error)

### Cause:

Wrong indentation in volumeMounts

### Fix:

Ensure correct YAML structure

---

## ❌ Pod not starting

```bash
kubectl describe pod <pod-name> -n mysql
```

👉 Always check **Events section**

---

# 🧠 Key Learnings

* StatefulSet = stable identity + storage
* Each pod gets its own PVC
* ConfigMap + Secret must match correctly
* Storage persists even after pod deletion
* Debugging is critical skill

---

# 🔄 Architecture Flow

```text
ConfigMap + Secret
        ↓
   StatefulSet
        ↓
       Pods
        ↓
       PVC
        ↓
        PV
        ↓
   Persistent Storage
```

---

# 📌 Final Result

* MySQL running with 3 pods
* Persistent storage attached
* Database (`devops`) created
* Verified inside container

---

## ⭐ If this helped, star the repo

