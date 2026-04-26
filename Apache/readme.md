---

# 🚀 Kubernetes HPA Hands-on (Apache Auto Scaling)

## 📌 Overview

This project demonstrates **Horizontal Pod Autoscaling (HPA)** in Kubernetes using an Apache web server.

The application automatically scales pods based on CPU utilization by generating real-time load using a BusyBox container.

---

## 🧱 Tech Stack

* Kubernetes (Kind / Any cluster)
* Docker (Apache httpd image)
* kubectl
* BusyBox (for load testing)

---

## 📂 Project Workflow (What You Did)

```id="k1t7v9"
Create Namespace → Deploy Apache → Expose Service → Test App → Enable HPA → Generate Load → Observe Auto Scaling
```

---

# ⚙️ Step-by-Step Implementation

---

## 1️⃣ Create Project Directory

```bash id="z9c6x2"
mkdir Apache
cd Apache
```

---

## 2️⃣ Create Namespace

```bash id="s4t9rm"
vim namespace.yml
kubectl apply -f namespace.yml
```

### namespace.yml

```yaml id="2q6b7p"
apiVersion: v1
kind: Namespace
metadata:
  name: apache
```

---

## 3️⃣ Create Apache Deployment

```bash id="8f9w2k"
vim deployment.yml
kubectl apply -f deployment.yml
```

### deployment.yml

```yaml id="9w2k4p"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-deployment
  namespace: apache
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: apache
        image: httpd
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: 100m
          limits:
            cpu: 200m
```

---

## 4️⃣ Verify Deployment

```bash id="7l0m9x"
kubectl get all -n apache
```

---

## 5️⃣ Create Service

```bash id="3f8k2n"
vim service.yml
kubectl apply -f service.yml
```

### service.yml

```yaml id="1x8p4t"
apiVersion: v1
kind: Service
metadata:
  name: apache-service
  namespace: apache
spec:
  selector:
    app: apache
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

---

## 6️⃣ Test Service (Inside Cluster)

```bash id="6r2k1z"
curl http://apache-service.apache.svc.cluster.local
```

---

## 7️⃣ Access Service Locally (Port Forward)

```bash id="5v8t2a"
kubectl port-forward service/apache-service -n apache 82:80 --address=0.0.0.0
```

Open in browser:

```
http://localhost:82
```

---

## 8️⃣ Manual Scaling (Testing)

```bash id="2p7d4s"
kubectl scale deployment apache-deployment -n apache --replicas=3
```

---

## 9️⃣ Create HPA

```bash id="4h9k2m"
vim hpa.yml
kubectl apply -f hpa.yml
```

### hpa.yml

```yaml id="7m3k9v"
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: apache-hpa
  namespace: apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: apache-deployment
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 5
```

---

## 🔟 Check HPA

```bash id="8m2k1q"
kubectl get hpa -n apache
```

Example output:

```id="n8z3k1"
cpu: 101%/5% → scaling triggered
```

---

## 1️⃣1️⃣ Generate Load (Important Step)

```bash id="7p2v6m"
kubectl run -i --tty load-generator --image=busybox -n apache -- /bin/sh
```

Inside container:

```sh id="6r4t9p"
while true; do wget -q -O- http://apache-service.apache.svc.cluster.local; done
```

---

## 1️⃣2️⃣ Observe Auto Scaling

```bash id="3m9k2x"
kubectl get hpa -n apache -w
```

```bash id="9k2m7v"
kubectl get pods -n apache
```

---

## 📊 Actual Result (Your Case)

* CPU usage increased → **101%**
* HPA scaled pods:

  * 1 → 4 → 5 (max limit)
* Even at 22–30% CPU → stayed at 5 pods (target was 5%)

---

## 🛑 Stop Load

```bash id="2m9v4k"
CTRL + C
kubectl delete pod load-generator -n apache
```

---

# 🧠 Key Concepts Learned

* Kubernetes Deployment & Service
* DNS-based service access
* Port forwarding
* Manual vs Auto scaling
* HPA working with CPU metrics
* Load testing inside cluster

---

# ⚠️ Important Requirements

### ✅ Metrics Server must be running

```bash id="8n2k4p"
kubectl top pods -n apache
```

---

### ✅ CPU requests must be defined

```yaml id="3p9k2m"
resources:
  requests:
    cpu: 100m
```

---

# 📈 How HPA Works

* Compares:

  ```
  Current CPU vs Target CPU
  ```
* If higher → scale up
* If lower → scale down

---

# 🎯 Final Output

✔ Apache app deployed
✔ Service exposed
✔ Load generated
✔ Pods auto-scaled (1 → 5)

---

# 🧑‍💻 Author

SAM (DevOps Learner)

---

# ⭐ If you like this project

Give it a ⭐ and share feedback!

---

