# Kubernetes Storage Configuration Lab

This README file provides a step-by-step guide to configuring persistent storage in Kubernetes using Persistent Volume Claims (PVCs). It also demonstrates how to ensure data persistence across pod deletions.

## Steps

### 1. Create an Nginx Deployment
- Use the following YAML to create a deployment with 1 replica:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### 2. Create a File Inside the Pod
- Execute into the pod and create a file at `/usr/share/nginx/html/hello.txt` with the content "hello iValve".
- Verify the file is served using `curl`.

### 3. Delete the Pod and Verify File Loss
- Delete the pod and wait for a new one to be created.
- Verify that the file is not present in the new pod.

### 4. Create a Persistent Volume Claim (PVC)
- Use the following YAML to create a PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 5. Attach the PVC to the Deployment
- Modify the deployment to mount the PVC at `/usr/share/nginx/html`:

```yaml
volumes:
- name: nginx-storage
  persistentVolumeClaim:
    claimName: nginx-pvc
```

### 6. Verify Data Persistence
- Repeat the file creation and pod deletion steps.
- Verify that the file persists across pod deletions.

## Comparison Between PV, PVC, and StorageClass

- **Persistent Volume (PV):**
  - A storage resource provisioned in the cluster.
  - Can be manually or dynamically provisioned.

- **Persistent Volume Claim (PVC):**
  - A request for storage by a user.
  - Binds to a PV to provide storage to a pod.

- **StorageClass:**
  - Allows dynamic provisioning of PVs.
  - Defines the type of storage (e.g., SSD, HDD) and provisioning policies.

## Cleanup

To delete all resources created in this lab, save the following YAML in a file named `cleanup.yaml` and run `kubectl delete -f cleanup.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-deployment
```




