---

# 🚀 Kubernetes Autoscaling Hands-on (HPA + VPA)

## 📌 Overview

This project demonstrates both **Horizontal Pod Autoscaler (HPA)** and **Vertical Pod Autoscaler (VPA)** in Kubernetes using an Apache web server.

* **HPA** → scales number of pods based on CPU
* **VPA** → adjusts CPU/Memory of existing pods

The project includes real-time load testing and observation of scaling behavior.

---

## 🧱 Tech Stack

* Kubernetes (Kind / Minikube / Any cluster)
* Docker (Apache httpd image)
* kubectl
* BusyBox (for load testing)
* Metrics Server

---

## 📂 Project Workflow

```bash
Create Namespace → Deploy Apache → Expose Service → Test App → HPA Scaling → Remove HPA → Setup VPA → Generate Load → Observe Resource Optimization
```

---

# ⚙️ PART 1: Horizontal Pod Autoscaler (HPA)

---

## 1️⃣ Create Namespace

```bash
kubectl apply -f namespace.yml
```

---

## 2️⃣ Deploy Apache

```bash
kubectl apply -f deployment.yml
kubectl apply -f service.yml
```

---

## 3️⃣ Access Application

```bash
kubectl port-forward service/apache-service -n apache 82:80 --address=0.0.0.0
```

Open:

```
http://localhost:82
```

---

## 4️⃣ Create HPA

```bash
kubectl apply -f hpa.yml
```

Check:

```bash
kubectl get hpa -n apache
```

---

## 5️⃣ Generate Load

```bash
kubectl run -i --tty load-generator --image=busybox -n apache -- /bin/sh
```

Inside container:

```sh
while true; do wget -q -O- http://apache-service.apache.svc.cluster.local; done
```

---

## 6️⃣ Observe Scaling

```bash
kubectl get pods -n apache
kubectl get hpa -n apache -w
```

### 📊 Result

* CPU reached ~100%
* Pods scaled: **1 → 5 (max)**

---

# ⚙️ PART 2: Vertical Pod Autoscaler (VPA)

---

## ⚠️ Step 1: Remove HPA

```bash
kubectl delete -f hpa.yml
```

> HPA and VPA should not control the same deployment simultaneously.

---

## 2️⃣ Install VPA

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
```

---

## 3️⃣ Apply VPA Configuration

```bash
kubectl apply -f vpa.yml
```

Check:

```bash
kubectl get vpa -n apache
```

Example:

```
NAME         MODE   CPU   MEM
apache-vpa   Auto   25m   250Mi
```

---

## 4️⃣ Generate Load Again

```bash
kubectl run -i --tty load-generator --image=busybox -n apache /bin/sh
```

```sh
while true; do wget -q -O- http://apache-service.apache.svc.cluster.local; done
```

---

## 5️⃣ Monitor Resource Usage

```bash
kubectl top pods -n apache
```

---

## 6️⃣ Observe VPA Recommendations

```bash
watch kubectl get vpa -n apache
```

### 📊 Result

* CPU usage increased
* VPA updated recommendation:

```
Before → 25m
After  → 126m
```

---

# 🧠 Key Concepts

| Feature        | Description                |
| -------------- | -------------------------- |
| HPA            | Scales number of pods      |
| VPA            | Adjusts CPU/Memory of pods |
| Metrics Server | Provides resource metrics  |
| BusyBox        | Used for load testing      |

---

# ⚠️ Important Notes

### ❗ VPA Warning

```
UpdateMode "Auto" is deprecated
```

Recommended modes:

* `Recreate`
* `Initial`
* `InPlaceOrRecreate`

---

### ❗ Requirements

```bash
kubectl top pods -n apache
```

✔ Metrics Server must be installed
✔ CPU requests must be defined

---

# 📈 HPA vs VPA

| Feature      | HPA            | VPA                   |
| ------------ | -------------- | --------------------- |
| Scaling Type | Horizontal     | Vertical              |
| Changes      | Pods count     | CPU/Memory            |
| Use Case     | Traffic spikes | Resource optimization |

---

# 🎯 Final Outcome

✔ Apache deployed successfully
✔ HPA scaled pods (1 → 5)
✔ VPA adjusted CPU dynamically (25m → 126m)
✔ Real-time load testing validated both approaches

---

# 🧑‍💻 Author

SAM (DevOps Learner)

---

# ⭐ If you like this project

Give it a ⭐ and share feedback

