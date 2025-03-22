# Jenkins Deployment with Kubernetes and Minikube

This guide details the process of deploying Jenkins using Kubernetes and Minikube, including creating a deployment with an init container, readiness and liveness probes, and exposing Jenkins with a NodePort service.

---

### Deploy Jenkins with Init Container
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      initContainers:
      - name: init-sleep
        image: busybox
        command: ["sh", "-c", "sleep 30"]
      containers:
      - name: jenkins
        image: jenkins/jenkins:lts
        ports:
        - containerPort: 8080
        livenessProbe:
          httpGet:
            path: /login
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /login
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

### Create NodePort Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: jenkins-service
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30000
  selector:
    app: jenkins
```

### Apply Configuration
```bash
kubectl apply -f jenkins-deployment.yaml
kubectl apply -f jenkins-service.yaml
```

---

## Verify Deployment

### Check Pods and Services
```bash
kubectl get pods
kubectl get svc
```

### Access Jenkins
```bash
minikube service jenkins-service --url
```
Use the URL provided to access Jenkins in your browser.

### Retrieve Initial Admin Password
```bash
kubectl exec -it <pod-name> -- cat /var/jenkins_home/secrets/initialAdminPassword
```
Replace `<pod-name>` with the name of your Jenkins pod.

---

## Comparison: Readiness vs. Liveness Probes
| **Feature**            | **Readiness Probe**                                   | **Liveness Probe**                                  |
|------------------------|------------------------------------------------------|---------------------------------------------------|
| **Purpose**            | Checks if the application is ready to serve traffic. | Checks if the application is still running.       |
| **Action on Failure**  | Excludes the pod from service routing.               | Restarts the container.                           |
| **Use Case**           | For delayed start or temporary unavailability.       | For detecting deadlocks or crashes.               |

---

## Comparison: Init vs. Sidecar Containers
| **Feature**            | **Init Container**                                   | **Sidecar Container**                              |
|------------------------|-----------------------------------------------------|--------------------------------------------------|
| **Purpose**            | Runs a task before the main container starts.       | Runs alongside the main container.               |
| **Lifetime**           | Runs once, then terminates.                         | Runs throughout the lifecycle of the pod.        |
| **Use Case**           | Dependency initialization or setup tasks.           | Auxiliary tasks like logging or monitoring.      |

---









