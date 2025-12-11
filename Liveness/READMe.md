🚀 Kubernetes Liveness Probe — Failure & Auto-Recovery Demo

A complete demonstration of how Kubernetes Liveness Probes work using a microservice that:

Starts healthy

Becomes unhealthy after 30 seconds

Triggers livenessProbe failures

Forces Kubernetes to restart the container repeatedly

This shows Kubernetes’ self-healing ability in a clear, practical way.

📌 What This Example Demonstrates

✔ How to create a /health endpoint
✔ How to simulate application failure after 30 seconds
✔ How Kubernetes detects failures
✔ How kubelet restarts containers automatically
✔ How to debug probe failures using logs & events

Very useful for DevOps learning, debugging, and interviews.

🧩 1. Application (Python Flask) — Healthy → Unhealthy Logic

The application:

Runs normally for 30 seconds

After that, returns HTTP 500 on /health

Kubernetes restarts the container using the liveness probe

app.py
<img width="483" height="496" alt="flask" src="https://github.com/user-attachments/assets/80e4150d-3a17-435e-9c4a-ab96dad670a0" />
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
    return "OK", 200              # Liveness passes

@app.route("/")
def home():
    return "App is running"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

🐳 2. Dockerfile
<img width="363" height="244" alt="dockerfile" src="https://github.com/user-attachments/assets/00817814-4481-4025-909c-79561dc85106" />
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

☸️ 3. Kubernetes Deployment With Liveness Probe

This probe hits /health every 5 seconds.
Once it receives 500, Kubernetes restarts the container.

<img width="574" height="658" alt="yml" src="https://github.com/user-attachments/assets/fe119237-b97c-4b62-91c2-ccab2f3f2a7f" />
apiVersion: apps/v1
kind: Deployment
metadata:
  name: liveness-demo

spec:
  replicas: 1
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
        image: <your-dockerhub-user>/livenessprobe:v1

        ports:
        - containerPort: 5000

        livenessProbe:
          httpGet:
            path: /health
            port: 5000

          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 2
          failureThreshold: 1

🔥 4. Deploy & Test
Apply deployment:
kubectl apply -f liveness-demo.yaml

Watch pod restart repeatedly:
kubectl get pods -w


After 30 seconds, the restarting will begin.

📝 5. Checking Pod Status
Describe pod to view events:
kubectl describe pod <pod-name>


You should see output like this:

<img width="944" height="829" alt="1" src="https://github.com/user-attachments/assets/ac1fcce1-d8ae-4ee5-8c49-415c62ce9171" /> <img width="945" height="174" alt="2" src="https://github.com/user-attachments/assets/c2f72310-dfd1-499a-ba6b-cbb51c718d12" /> <img width="964" height="199" alt="3" src="https://github.com/user-attachments/assets/d62b5053-accc-43d6-8f33-0a9068b5a413" />
These logs clearly show:

Liveness probe failed

Failure code: 500

Kubelet restarted the container

📄 6. Why Kubernetes Keeps Restarting the Container

Once the app becomes unhealthy:

1️⃣ /health returns 500
2️⃣ Liveness probe fails
3️⃣ Kubernetes kills the container
4️⃣ Kubernetes restarts the container
5️⃣ App runs healthy again for 30 sec
6️⃣ App becomes unhealthy again

🔁 This loop continues forever → confirming the probe works correctly.

🎯 7. What You Learn From This Example

✔ How liveness probes detect failures
✔ How Kubernetes self-heals broken apps
✔ How to design /health endpoints
✔ How to debug pod events and logs
✔ Why liveness ≠ readiness ≠ startup probes

🧠 Key Concept Summary
Feature	Purpose
Liveness Probe	Detects if the app is alive. If fails → restart container.
Readiness Probe	Detects if app is ready to serve traffic. If fails → remove from Service endpoints.
Startup Probe	Prevents premature restarts for slow-boot apps.
⭐ Final Notes

This setup is extremely useful for:

Understanding Kubernetes internals

DevOps interview preparation

Learning self-healing microservice design

Hands-on Kubernetes debugging
