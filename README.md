## ✅ Project: Full Stack App (Flask + PostgreSQL) on AKS

---

### 📁 Project Structure

```
aks-flask-postgres/
│
├── app/
│   ├── app.py
│   ├── requirements.txt
│   └── Dockerfile
│
├── k8s/
│   ├── namespace.yaml
│   ├── postgres-deployment.yaml
│   ├── postgres-service.yaml
│   ├── app-deployment.yaml
│   ├── app-service.yaml
│   └── secret.yaml
│
├── .gitignore
└── README.md
```

---

## 🧱 Step 1: Flask Application (`app/app.py`)

```python
from flask import Flask
import psycopg2
import os

app = Flask(__name__)

@app.route('/')
def hello():
    try:
        conn = psycopg2.connect(
            host=os.environ.get('DB_HOST'),
            database=os.environ.get('POSTGRES_DB'),
            user=os.environ.get('POSTGRES_USER'),
            password=os.environ.get('POSTGRES_PASSWORD')
        )
        cur = conn.cursor()
        cur.execute("SELECT version();")
        db_version = cur.fetchone()
        cur.close()
        conn.close()
        return f"Hello from Flask! DB version: {db_version}"
    except Exception as e:
        return f"Database connection failed: {str(e)}"
```

---

## 📦 `requirements.txt`

```
flask
psycopg2
```

---

## 🐳 Dockerfile for Flask

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

---

## 🔐 Kubernetes Secret (`k8s/secret.yaml`)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: pg-secret
type: Opaque
data:
  POSTGRES_USER: cG9zdGdyZXM=       # postgres (base64)
  POSTGRES_PASSWORD: cGFzc3dvcmQ=   # password (base64)
  POSTGRES_DB: dGVzdGRi             # testdb (base64)
```

---

## 🐘 PostgreSQL Deployment (`k8s/postgres-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:14
          env:
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: pg-secret
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: pg-secret
                  key: POSTGRES_PASSWORD
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: pg-secret
                  key: POSTGRES_DB
          ports:
            - containerPort: 5432
```

---

## 🛜 PostgreSQL Service (`k8s/postgres-service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  ports:
    - port: 5432
  selector:
    app: postgres
```

---

## 🚀 Flask App Deployment (`k8s/app-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
        - name: flask-app
          image: <your-dockerhub-username>/flask-app:latest
          ports:
            - containerPort: 5000
          env:
            - name: DB_HOST
              value: postgres
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: pg-secret
                  key: POSTGRES_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: pg-secret
                  key: POSTGRES_PASSWORD
            - name: POSTGRES_DB
              valueFrom:
                secretKeyRef:
                  name: pg-secret
                  key: POSTGRES_DB
```

---

## 📡 Flask App Service (`k8s/app-service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: LoadBalancer
  selector:
    app: flask-app
  ports:
    - port: 80
      targetPort: 5000
```

---

## 🗂 Namespace (Optional)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: flaskapp
```

---

## 🚀 Deployment Steps

### 1. 🔧 Prerequisites

* Azure CLI installed
* Docker installed and configured
* AKS cluster created
* Kubectl installed

---

### 2. 🔨 Build & Push Docker Image

```bash
cd app
docker build -t <your-dockerhub-username>/flask-app:latest .
docker push <your-dockerhub-username>/flask-app:latest
```

---

### 3. 🔗 Connect to AKS Cluster

```bash
az aks get-credentials --resource-group <your-rg> --name <your-aks-cluster>
```

---

### 4. 📥 Apply Kubernetes Config

```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secret.yaml -n flaskapp
kubectl apply -f k8s/postgres-deployment.yaml -n flaskapp
kubectl apply -f k8s/postgres-service.yaml -n flaskapp
kubectl apply -f k8s/app-deployment.yaml -n flaskapp
kubectl apply -f k8s/app-service.yaml -n flaskapp
```

---

### 5. 🌐 Access Application

```bash
kubectl get svc -n flaskapp
```

Use the **EXTERNAL-IP** of the `flask-service` to access your app in a browser.

---

## 📘 Optional Enhancements

* Add Helm chart for templating.
* Use Azure Container Registry instead of Docker Hub.
* Add ingress controller like NGINX.
* Connect to Azure Database for PostgreSQL (instead of container).

---

