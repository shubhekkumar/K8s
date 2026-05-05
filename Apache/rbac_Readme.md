---

# 🔐 Kubernetes RBAC Hands-on (Role-Based Access Control)

## 📌 Overview

This project demonstrates **Role-Based Access Control (RBAC)** in Kubernetes using a practical setup with:

* **Role**
* **ServiceAccount**
* **RoleBinding**
* Access verification using `kubectl auth can-i`

The goal is to control **who can access what resources inside a namespace**.

---

## 🧱 Tech Stack

* Kubernetes (Kind / Minikube / Any Cluster)
* kubectl
* YAML manifests

---

## 📂 Project Structure

```bash
.
├── namespace.yml
├── role.yml
├── role_binding.yml
├── service_account.yml
```

---

## ⚙️ Setup & Execution

### 1️⃣ Create Namespace

```bash
kubectl apply -f namespace.yml
```

---

### 2️⃣ Create ServiceAccount

```bash
kubectl apply -f service_account.yml
```

---

### 3️⃣ Create Role

```bash
kubectl apply -f role.yml
```

---

### 4️⃣ Bind Role to ServiceAccount

```bash
kubectl apply -f role_binding.yml
```

---

## 🔑 RBAC Configuration

### 🔹 Role

Defines permissions inside namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: apache-manager
  namespace: apache
rules:
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "services"]
  verbs: ["get", "list", "create", "delete", "patch"]
```

---

### 🔹 ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: apache-user
  namespace: apache
```

---

### 🔹 RoleBinding

Connects Role → ServiceAccount:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: apache-manager-rolebinding
  namespace: apache

subjects:
- kind: ServiceAccount
  name: apache-user
  namespace: apache

roleRef:
  kind: Role
  name: apache-manager
  apiGroup: rbac.authorization.k8s.io
```

---

## 🧪 Testing Access (Important)

### ❌ Incorrect Way

```bash
kubectl auth can-i get pods --as=apache-user -n apache
```

➡️ Result: `no`
Reason: Kubernetes treats this as a **normal user**, not a ServiceAccount.

---

### ✅ Correct Way

```bash
kubectl auth can-i get pods \
--as=system:serviceaccount:apache:apache-user \
-n apache
```

➡️ Result: `yes`

---

## 🔍 Verify Permissions

```bash
kubectl auth can-i --list \
--as=system:serviceaccount:apache:apache-user \
-n apache
```

---

## 🐞 Debugging Issues Faced

### 1. Typo in Role Name

* `apche-manager` ❌
* `apache-manager` ✅

---

### 2. Wrong Subject Type

```yaml
kind: User ❌
kind: ServiceAccount ✅
```

---

### 3. Wrong Resource Names

```yaml
"pod", "deployment", "service" ❌
"pods", "deployments", "services" ✅
```

---

### 4. Invalid Verb

```yaml
"apply" ❌
```

Correct verbs:

```yaml
get, list, create, delete, patch
```

---

## 📊 Key Learnings

* RBAC = **Who can do what on which resource**
* ServiceAccounts require full identity:

  ```
  system:serviceaccount:<namespace>:<name>
  ```
* RBAC is enforced by **kube-apiserver**
* Always use **plural resource names**
* Follow **Principle of Least Privilege**

---

## 🚀 Real DevOps Use Case

* Restrict CI/CD pipelines to specific namespaces
* Limit developers to only:

  * view logs
  * deploy apps
* Prevent unauthorized cluster-wide access

---

## 🧠 Interview Ready Points

* Difference between:

  * Role vs ClusterRole
  * RoleBinding vs ClusterRoleBinding
* RBAC flow:

  ```
  User/ServiceAccount → Role → RoleBinding → Resource
  ```
* Common mistakes:

  * Wrong identity (`--as`)
  * Wrong resource names
  * Namespace mismatch

---

## 📌 Conclusion

This hands-on demonstrates how RBAC ensures **secure, controlled access** in Kubernetes and is a **core skill for DevOps engineers**.

---

