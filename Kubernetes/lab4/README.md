#  Security and RBAC - Accessing Pods Using a Service Account

This lab demonstrates how to create a Service Account, define a Role, bind the Role to the Service Account, and use the Service Account to access Pods in Kubernetes.

## Steps

### 1. Create a Service Account
Create a Service Account named `my-service-account`:

```bash
kubectl create serviceaccount my-service-account
```

### 2. Create a Role (pod-reader)
Create a Role named `pod-reader` that allows read-only access to Pods in the default namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

Apply the Role:

```bash
kubectl apply -f pod-reader-role.yaml
```

### 3. Bind the Role to the Service Account
Create a RoleBinding to bind the `pod-reader` Role to `my-service-account`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply the RoleBinding:

```bash
kubectl apply -f pod-reader-binding.yaml
```

### 4. Generate a Token for the Service Account
Generate a token for the Service Account:

```bash
kubectl create token my-service-account
```

### 5. Access Pods Using the Service Account
Use the token to access Pods in the default namespace:

```bash
kubectl get pods --token=<service-account-token>
```

### 6. Verify Permissions
Verify that the Service Account has the required permissions:

```bash
kubectl auth can-i get pods --as=system:serviceaccount:default:my-service-account
```

### 7. Create a Test Pod (Optional)
Create a test Pod to verify access:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply the Pod:

```bash
kubectl apply -f test-pod.yaml
```

### 8. Cleanup
Delete the resources created in this lab:

```bash
kubectl delete pod test-pod
kubectl delete rolebinding pod-reader-binding
kubectl delete role pod-reader
kubectl delete serviceaccount my-service-account
```

## Comparison Between RBAC Components

| Term               | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| **Service Account**| An account used by Pods to interact with the Kubernetes API.               |
| **Role**           | A set of permissions applied to specific resources within a namespace.     |
| **RoleBinding**    | Binds a Role to a Service Account, User, or Group within a namespace.       |
| **ClusterRole**    | A set of permissions applied to resources cluster-wide.                    |
| **ClusterRoleBinding** | Binds a ClusterRole to a Service Account, User, or Group cluster-wide. |






