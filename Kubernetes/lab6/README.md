# Storing Data & Setting Resource Limits

This project demonstrates how to create a Namespace, apply a Resource Quota, deploy MySQL with resource requests and limits, and use ConfigMap and Secret to store configuration data and passwords.

## Prerequisites

- **kubectl**: To manage the Kubernetes cluster.
- **Minikube**: To run Kubernetes locally.
- **Docker**: To run MySQL containers.

## Steps to Execute

### 1. Create a Namespace

1. Create a new Namespace:
   ```bash
   kubectl create namespace ivolve
   ```
2. Verify the Namespace creation:
   ```bash
   kubectl get namespaces
   ```

### 2. Apply a Resource Quota

Create a `resource-quota.yaml` file:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-quota
  namespace: ivolve
spec:
  hard:
    pods: "10"  # Maximum number of Pods in the Namespace
```

Apply the Resource Quota:

```bash
kubectl apply -f resource-quota.yaml
```

Verify the Resource Quota:

```bash
kubectl get resourcequota -n ivolve
```

### 3. Deploy MySQL

Create a `mysql-deployment.yaml` file:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  namespace: ivolve
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: mysql-config
              key: database
        - name: MYSQL_USER
          valueFrom:
            configMapKeyRef:
              name: mysql-config
              key: user
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: user-password
        resources:
          requests:
            cpu: "0.5"
            memory: "1Gi"
          limits:
            cpu: "1"
            memory: "2Gi"
        ports:
        - containerPort: 3306
```

Apply the Deployment:

```bash
kubectl apply -f mysql-deployment.yaml
```

Verify the Pod creation:

```bash
kubectl get pods -n ivolve
```

### 4. Create a ConfigMap

Create a `mysql-configmap.yaml` file:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
  namespace: ivolve
data:
  database: "mydatabase"
  user: "myuser"
```

Apply the ConfigMap:

```bash
kubectl apply -f mysql-configmap.yaml
```

Verify the ConfigMap:

```bash
kubectl get configmap -n ivolve
```

### 5. Create a Secret

Create a `mysql-secret.yaml` file:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
  namespace: ivolve
type: Opaque
data:
  root-password: cm9vdHBhc3N3b3Jk  # base64 encoded "rootpassword"
  user-password: dXNlcnBhc3N3b3Jk  # base64 encoded "userpassword"
```

Apply the Secret:

```bash
kubectl apply -f mysql-secret.yaml
```

Verify the Secret:

```bash
kubectl get secret -n ivolve
```

### 6. Access the Pod and Verify the Database Configuration

1. Get the Pod name:
   ```bash
   kubectl get pods -n ivolve
   ```
2. Access the Pod:
   ```bash
   kubectl exec -it <pod-name> -n ivolve -- /bin/bash
   ```
3. Log in to MySQL:
   ```bash
   mysql -u root -p
   ```
   Enter the root password .

4. Verify the database:
   ```sql
   SHOW DATABASES;
   ```

5. Verify the user:
   ```sql
   SELECT User FROM mysql.user;
   ```



