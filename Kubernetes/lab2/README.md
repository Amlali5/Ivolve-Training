# Lab 27: Deployment vs. StatefulSet

This lab explains the differences between **Deployment** and **StatefulSet** in Kubernetes. It includes creating a MySQL StatefulSet with 3 replicas and a corresponding Service.

---

## **Comparison: Deployment vs. StatefulSet**

| **Aspect**            | **Deployment**                                                                         | **StatefulSet**                                                                       |
|------------------------|-----------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| **Use Case**           | Ideal for stateless applications (e.g., web servers, API services).                     | Ideal for stateful applications requiring stable storage and network identity (e.g., databases). |
| **Pod Management**     | Pods are interchangeable with random names (e.g., `nginx-12345-abcde`).                 | Pods have unique and stable identities (e.g., `mysql-0`, `mysql-1`, `mysql-2`).       |
| **Storage**            | Works well with ephemeral storage or shared storage.                                   | Provides dedicated persistent storage for each Pod, ensuring data consistency.        |
| **Scaling**            | Easy to scale up or down dynamically without dependencies.                              | Scaling requires careful coordination to maintain data integrity and stability.       |
| **Rolling Updates**    | Efficient for rolling updates, ensuring zero downtime for stateless apps.               | Supports ordered and controlled rolling updates, preserving Pod identity and data.    |
| **Networking**         | Pods share a common service name but no guaranteed Pod-to-Pod consistency.              | Ensures stable network identity and DNS for each Pod, critical for clustered apps.    |

---

## **Steps to Complete the Lab**

### **1. Create a MySQL StatefulSet with 3 Replicas**

1. Create the `mysql-statefulset.yaml` file:
   ```yaml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: mysql
   spec:
     serviceName: "mysql"
     replicas: 3
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
           - name: MYSQL_ROOT_PASSWORD
             value: "password"
           ports:
           - containerPort: 3306
           volumeMounts:
           - name: mysql-persistent-storage
             mountPath: /var/lib/mysql
     volumeClaimTemplates:
     - metadata:
         name: mysql-persistent-storage
       spec:
         accessModes: [ "ReadWriteOnce" ]
         resources:
           requests:
             storage: 1Gi
   ```

   Apply the StatefulSet:
   ```bash
   kubectl apply -f mysql-statefulset.yaml
   ```

   Verify the StatefulSet:
   ```bash
   kubectl get statefulsets
   ```

   Verify the Pods:
   ```bash
   kubectl get pods
   ```

   Expected output: 3 Pods named `mysql-0`, `mysql-1`, `mysql-2`.

2. Create the Service for the MySQL StatefulSet

   Create the `mysql-service.yaml` file:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: mysql
   spec:
     selector:
       app: mysql
     ports:
     - port: 3306
       targetPort: 3306
     clusterIP: None
   ```

   Apply the Service:
   ```bash
   kubectl apply -f mysql-service.yaml
   ```

   Verify the Service:
   ```bash
   kubectl get svc
   ```

3. Verify Persistent Storage

   Check Persistent Volume Claims (PVCs):
   ```bash
   kubectl get pvc
   ```

   Expected output: 3 PVCs named `mysql-persistent-storage-mysql-0`, `mysql-persistent-storage-mysql-1`, `mysql-persistent-storage-mysql-2`.

   Check Persistent Volumes (PVs):
   ```bash
   kubectl get pv
   ```

4. Clean Up (Optional)

   Delete the StatefulSet and Service:
   ```bash
   kubectl delete -f mysql-statefulset.yaml
   kubectl delete -f mysql-service.yaml
   ```




