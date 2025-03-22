# Updating Applications and Rolling Back Changes

This lab demonstrates how to deploy an application, update it, and roll back changes in a Kubernetes cluster. The tasks include deploying NGINX, exposing it via a service, updating the image to Apache, and rolling back to the previous version.

---

## **Steps to Complete the Lab**

### **1. Deploy NGINX with 3 Replicas**

#### **Create and Apply the Deployment**
1. Create a file named `nginx-deployment.yaml` with the following content:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx
   spec:
     replicas: 3
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

2. Apply the deployment:
   ```bash
   kubectl apply -f nginx-deployment.yaml
   ```

3. Verify the pods:
   ```bash
   kubectl get pods
   ```

#### **Create and Apply the Service**
1. Create a file named `nginx-service.yaml` with the following content:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx
   spec:
       selector:
       app: nginx
       ports:
       - protocol: TCP
         port: 80
         targetPort: 80
   ```

2. Apply the service:
   ```bash
   kubectl apply -f nginx-service.yaml
   ```

3. Verify the service:
   ```bash
   kubectl get svc
   ```

#### **Access NGINX Locally**
1. Start port forwarding:
   ```bash
   kubectl port-forward svc/nginx 8080:80
   ```

2. Open a browser and navigate to `http://localhost:8080` or use the following command:
   ```bash
   curl http://localhost:8080
   ```

---

### **2. Update NGINX Image to Apache**

1. Update the deployment image to Apache:
   ```bash
   kubectl set image deployment/nginx nginx=httpd:latest
   ```

2. Verify the pods:
   ```bash
   kubectl get pods
   ```

3. Access the application to confirm the update:
   ```bash
   curl http://localhost:8080
   ```

---

### **3. View Deployment's Rollout History**

1. View the deployment's rollout history:
   ```bash
   kubectl rollout history deployment/nginx
   ```

---

### **4. Roll Back to the Previous Version**

1. Roll back to the previous image version:
   ```bash
   kubectl rollout undo deployment/nginx
   ```

2. Monitor the pod status during the rollback:
   ```bash
   kubectl get pods -w
   ```

3. Access the application again to confirm the rollback:
   ```bash
   curl http://localhost:8080
   ```

---

### **5. Clean Up Resources (Optional)**

1. Delete the deployment and service:
   ```bash
   kubectl delete -f nginx-deployment.yaml
   kubectl delete -f nginx-service.yaml
   ```

---

## **Commands Summary**

```bash
# Apply deployment
kubectl apply -f nginx-deployment.yaml

# Apply service
kubectl apply -f nginx-service.yaml

# Port forwarding
kubectl port-forward svc/nginx 8080:80

# Update image to Apache
kubectl set image deployment/nginx nginx=httpd:latest

# View rollout history
kubectl rollout history deployment/nginx

# Roll back to previous version
kubectl rollout undo deployment/nginx

# Monitor pods
kubectl get pods -w

# Clean up
kubectl delete -f nginx-deployment.yaml
kubectl delete -f nginx-service.yaml
```

---

## **Expected Results**

- **NGINX Deployment:** Successfully deployed with 3 replicas.
- **Service Exposure:** Accessible locally via port forwarding.
- **Update to Apache:** Image updated successfully.
- **Rollback:** Reverted to the original NGINX image.
- **Learning Outcome:** Hands-on experience with Kubernetes deployment, updates, and rollbacks.





