# Exercise 2: Deploy a Flask App on Minikube using kubectl and YAML

## Objective
Learn Kubernetes basics using Minikube to set up a single-node cluster and deploy a Python Flask application.

## Files in this directory
| File | Description |
|------|-------------|
| `app.py` | Flask application that returns "Hello from Flask on Kubernetes!" |
| `Dockerfile` | Docker image definition for the Flask app |
| `flask-deployment.yaml` | Kubernetes Deployment + Service YAML file |

## Steps Performed

### Step 1: Start Minikube
```bash
minikube start
```

### Step 2: Create the Flask Application
Created `app.py` — a simple Flask app listening on port 15000.

### Step 3: Create the Dockerfile
Created `Dockerfile` using `python:3.8-slim` as base image.

### Step 4: Build Docker Image with Minikube's Docker Daemon
```bash
# Display Minikube Docker environment variables
minikube docker-env

# Set Docker CLI to use Minikube's Docker daemon
eval $(minikube docker-env)

# Build the image
docker build -t flask-app .
```

### Step 5: Create the Kubernetes Deployment YAML
Created `flask-deployment.yaml` with:
- **Deployment**: 1 replica of flask-app container with `imagePullPolicy: Never`
- **Service**: NodePort service exposing port 15000

> **Note on imagePullPolicy: Never**  
> This tells Kubernetes to only use the local Docker image (built in Minikube's Docker daemon) and NOT pull from Docker Hub.

### Step 6: Deploy the Application
```bash
kubectl apply -f flask-deployment.yaml
```

**Output:**
```
deployment.apps/flask-app created
service/flask-app-service created
```

### Step 7: Check Deployment Status
```bash
kubectl get deployments
```

**Output:**
```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
flask-app    1/1     1            1           5s
```

### Step 8: Verify Pods
```bash
kubectl get pods -l app=flask-app
```

**Output:**
```
NAME                          READY   STATUS    RESTARTS   AGE
flask-app-b8cd75b6f-tpdpr     1/1     Running   0          10s
```

### Step 9: Describe the Deployment
```bash
kubectl describe deployment flask-app
```

Key details from output:
- Replicas: 1 desired | 1 updated | 1 total | 1 available
- Image: flask-app:latest
- Port: 15000/TCP
- Strategy: RollingUpdate

### Step 10: View Deployment Logs
```bash
kubectl logs <pod-name>
```

**Output:**
```
 * Serving Flask app 'app'
 * Debug mode: off
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:15000
 * Running on http://10.0.0.210:15000
```

### Step 11: Check Services
```bash
kubectl get services
```

**Output:**
```
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP           7d
flask-app-service   NodePort    10.108.45.123   <none>        15000:31234/TCP   30s
```

### Step 12: Access the Flask Application
```bash
# Get the service URL
minikube service flask-app-service --url
```

**Output:**
```
http://127.0.0.1:36157
```

```bash
# Access the application
curl http://127.0.0.1:36157
```

**Output:**
```
Hello from Flask on Kubernetes!
```

✅ The Flask application is now accessible on Kubernetes!

## Key Concepts Learned

### Port vs TargetPort
- **port (15000)**: The port exposed by the Kubernetes Service (external access)
- **targetPort (15000)**: The port the container listens on (internal)

### Request Flow
```
External Request (port 15000) → Service (port 15000) → Container (port 15000)
```

### imagePullPolicy Options
| Policy | Behavior |
|--------|----------|
| `Always` | Always pulls from registry |
| `IfNotPresent` | Pulls only if not available locally |
| `Never` | Only uses local images |

## Q&A

**Q1: What is the purpose of `minikube service flask-app-service --url`?**  
A1: To provide the URL for accessing the flask-app-service running in Minikube.

**Q2: Why is `targetPort` used in Kubernetes Service configuration?**  
A2: To specify the port number on which the container is listening.

**Q3: What is the difference between `port` and `targetPort`?**  
A3: `port` is the exposed Service port, while `targetPort` is the container port.

**Q4: How do you access a Flask application running in Minikube?**  
A4: Use `minikube service <service-name> --url` to get the access URL.

**Q5: Why does the terminal need to remain open when using Docker driver on Linux?**  
A5: Because the Docker driver requires the terminal to stay open to maintain the Minikube connection.
