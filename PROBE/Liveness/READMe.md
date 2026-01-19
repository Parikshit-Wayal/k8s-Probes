# 🚀 Kubernetes Liveness Probe — Failure & Auto-Recovery Demo

A complete demonstration of how Kubernetes **Liveness Probes** work using a microservice that:

1.  Starts **healthy**
2.  Becomes **unhealthy** after 30 seconds
3.  Triggers **livenessProbe failures**
4.  Forces Kubernetes to **restart the container repeatedly**

This shows Kubernetes’ **self-healing ability** in a clear, practical way.

---

## 📌 What This Example Demonstrates

* ✔ How to create a `/health` endpoint
* ✔ How to simulate application failure after 30 seconds
* ✔ How Kubernetes detects failures
* ✔ How `kubelet` restarts containers automatically
* ✔ How to debug probe failures using logs & events

Very useful for DevOps learning, debugging, and interviews.

---

## 🧩 1. Application (Python Flask) — Healthy → Unhealthy Logic

The application:

* Runs normally for **30 seconds**
* After that, returns **HTTP 500** on `/health`
* Kubernetes restarts the container using the liveness probe

**`app.py`**
<img width="483" height="496" alt="flask" src="https://github.com/user-attachments/assets/80e4150d-3a17-435e-9c4a-ab96dad670a0" />
```python
from flask import Flask
import time

# Create Flask application
app = Flask(__name__)

# Capture start time
start_time = time.time()

@app.route("/health")
def health():
    """
    Liveness probe endpoint:
    - Returns 200 for first 30 seconds
    - Returns 500 afterwards -> Kubernetes will restart the container
    """
    if time.time() - start_time > 30:
        return "UNHEALTHY", 500   # Liveness fails
    return "OK", 200             # Liveness passes

@app.route("/")
def home():
    return "App is running"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

```
---

<img width="363" height="244" alt="dockerfile" src="https://github.com/user-attachments/assets/00817814-4481-4025-909c-79561dc85106" />

---
```
🐳 2. Dockerfile

Dockerfile

# Base image with Python
FROM python:3.9-slim

# Working directory

WORKDIR /app

# Copy application

COPY app.py .

# Install Flask

RUN pip install flask

# Start app

CMD ["python3", "app.py"]

```
---

```

☸️ 3. Kubernetes Deployment With Liveness Probe
apiVersion: apps/v1
kind: Deployment
metadata:
  name: liveness-demo   # Name of the deployment

spec:
  replicas: 1           # Number of pods to run
  selector:
    matchLabels:
      app: liveness-demo

  template:
    metadata:
      labels:
        app: liveness-demo

    spec:
      containers:
      - name: python-app
        image: parikshit1212/livenessprobe 
        # Replace above with your own image

        ports:
        - containerPort: 5000  # App runs on port 5000

        # ---------------- LIVENESS PROBE START ----------------
        livenessProbe:

          httpGet:
            path: /health      # Kubernetes will hit this endpoint
            port: 5000         # On container's port 5000

          initialDelaySeconds: 5
          # Wait 5 seconds before starting the first probe
          # Allows the container time to start the app

          periodSeconds: 5
          # Probe runs every 5 seconds

          timeoutSeconds: 2
          # Kubelet will wait max 2 seconds for a response

          failureThreshold: 1
          # If 1 failure occurs, restart the container immediately

        # ---------------- LIVENESS PROBE END ----------------


-----
```

## 🔥 4. Deploy & Test

1.  **Apply deployment:**
    ```bash
    kubectl apply -f liveness-demo.yaml
    ```
2.  **Watch pod restart repeatedly:**
    ```bash
    kubectl get pods -w
    ```

After 30 seconds, the restarting will begin.

-----

## 📝 5. Checking Pod Status

**Describe pod to view events:**

```bash
kubectl describe pod <pod-name>

```
You should see output like this:

<img width="944" height="829" alt="1" src="https://github.com/user-attachments/assets/9c06bf92-b269-427d-8af5-1ed5b2dfd2bb" />

<br>
<br>

<img width="945" height="174" alt="2" src="https://github.com/user-attachments/assets/eb860b61-218b-449b-a001-bb1225f2045b" />

<br>
<br>

<img width="964" height="199" alt="3" src="https://github.com/user-attachments/assets/66f08584-db50-45b3-8c05-1d6e8c9cfdb6" />

<br>
<br>


These contaner events clearly show:

  * Liveness probe **failed**
  * Failure code: **500**
  * Kubelet **restarted** the container

-----

## 📄 6. Why Kubernetes Keeps Restarting the Container

Once the app becomes unhealthy:

1.  `/health` returns **500**
2.  Liveness probe **fails**
3.  Kubernetes **kills** the container
4.  Kubernetes **restarts** the container
5.  App runs **healthy** again for **30 sec**
6.  App becomes **unhealthy** again

**🔁 This loop continues forever → confirming the probe works correctly.**


-----

