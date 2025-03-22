# Network Configuration

## Prerequisites

- **Docker**: To build the image and run containers.
- **Minikube**: To run Kubernetes locally.
- **kubectl**: To manage the Kubernetes cluster.

## Steps to Execute

### 1. Build the Docker Image

1. Clone the repository:
   ```bash
   git clone https://github.com/IbrahimAdell/App1.git
   cd App1
   ```
2. Build the image:
   ```bash
   docker build -t my-app-image:latest .
   ```
3. (Optional) Load the image into Minikube:
   ```bash
   minikube image load my-app-image:latest
   ```

### 2. Create a Deployment

Create a `deployment.yaml` file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app-image:latest
        ports:
        - containerPort: 80
```

Apply the Deployment:

```bash
kubectl apply -f deployment.yaml
```

### 3. Create a Service

Create a `service.yaml` file:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

Apply the Service:

```bash
kubectl apply -f service.yaml
```

### 4. Create an Ingress

Create an `ingress.yaml` file:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: my-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

Apply the Ingress:

```bash
kubectl apply -f ingress.yaml
```

### 5. Enable NGINX Ingress Controller

Enable the Ingress Controller in Minikube:

```bash
minikube addons enable ingress
```

### 6. Update the /etc/hosts File

Get the Minikube IP address:

```bash
minikube ip
```

Add the following line to the `/etc/hosts` file:

```
192.168.49.2    my-app.local
```

### 7. Test the Application

Open a browser and navigate to:

```
http://my-app.local
```

Or use curl to access the application:

```bash
curl http://my-app.local
```


