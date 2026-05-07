# 🚀 Kubernetes Dashboard + Application Deployment (From Scratch)

## 📌 Overview

This setup shows how to:

1. Configure and access Kubernetes Dashboard
2. Create an admin user for login
3. Deploy an application (nginx workloads)
4. View and verify everything inside the Dashboard

---

# ⚙️ Step 1: Install Kubernetes Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

This installs:

* Dashboard UI
* Metrics scraper
* Required RBAC roles

---

# 🔐 Step 2: Create Admin User

Create file: `dashboard-admin-user.yml`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user-binding
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
```

Apply:

```bash
kubectl apply -f dashboard-admin-user.yml
```

---

# 🔑 Step 3: Generate Login Token

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

Copy this token (used for login).

---

# 🌐 Step 4: Start Dashboard Access

```bash
kubectl proxy &
```

Open in browser:

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

Login using:

* Select **Token**
* Paste token

---

# 📦 Step 5: Deploy Application (nginx)

### Create Namespace

```bash
kubectl create namespace nginx
```

---

### Apply All Manifests

(inside your nginx folder)

```bash
kubectl apply -f .
```

Resources created:

* Pod
* Deployment
* ReplicaSet
* DaemonSet
* Job
* CronJob
* Ingress (after installing controller)

---

# 🌍 Step 6: Fix Ingress (if error occurs)

If you see webhook error → install ingress controller:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

Then reapply:

```bash
kubectl apply -f ingress.yml
```

---

# 📊 Step 7: Verify in Dashboard

In Dashboard:

1. Select namespace → `nginx`
2. Go to **Workloads**

You will see:

* Pods (Running)
* Deployments (Ready)
* DaemonSets (Running on all nodes)
* Jobs (Completed)
* ReplicaSets

---

# 🧠 Key Points

* Dashboard needs **ServiceAccount + ClusterRoleBinding**
* Token is required for login (expires, regenerate if needed)
* `kubectl proxy` exposes dashboard locally
* Namespace helps organize resources
* Ingress requires controller (not automatic)

---

# ✅ Outcome

* Dashboard successfully accessed
* Admin user created with full access
* Application deployed in `nginx` namespace
* All workloads visible and monitored in Dashboard

---

