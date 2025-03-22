# DaemonSet with Taint and Toleration

This guide demonstrates how to create a DaemonSet, apply Taints and Tolerations, and compare Taint & Toleration with Node Affinity.

## Complete Steps

### Step 1: Create a DaemonSet for Prometheus Node-Exporter

Create the `node-exporter-daemonset.yaml` file:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  labels:
    app: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
```

Apply the DaemonSet:

```bash
kubectl apply -f node-exporter-daemonset.yaml
```

Verify the DaemonSet:

```bash
kubectl get daemonset
kubectl get pods -o wide
```

You should see the `node-exporter` running on all nodes.

### Step 2: Taint the Node

Taint a specific node using the key-value pair `color=red`:

```bash
kubectl taint nodes <node-name> color=red:NoSchedule
```

Replace `<node-name>` with the correct node name (use `kubectl get nodes` to retrieve node names).

Verify the taint:

```bash
kubectl describe node <node-name>
```

Check the `Taints` section in the description.

### Step 3: Create a Pod with Toleration for `color=blue`

Create the `pod-with-toleration-blue.yaml` file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-blue
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "color"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

Apply the Pod:

```bash
kubectl apply -f pod-with-toleration-blue.yaml
```

Check the Pod status:

```bash
kubectl get pods
```

The Pod will be in `Pending` state as it does not tolerate the `color=red` taint.

### Step 4: Change Toleration to `color=red`

Create the `pod-with-toleration-red.yaml` file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod-red
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: "color"
    operator: "Equal"
    value: "red"
    effect: "NoSchedule"
```

Apply the Pod:

```bash
kubectl apply -f pod-with-toleration-red.yaml
```

Check the Pod status:

```bash
kubectl get pods
```

The Pod will be in `Running` state as it tolerates the `color=red` taint.

### Step 5: Compare Taint & Toleration and Node Affinity

| Feature           | Taint & Toleration                                  | Node Affinity                                   |
|-------------------|-----------------------------------------------------|------------------------------------------------|
| **Purpose**       | Prevent Pods from running on specific nodes unless tolerated. | Attract Pods to specific nodes based on labels. |
| **Behavior**      | Repels Pods unless they have matching tolerations.  | Attracts Pods to nodes matching specified labels. |
| **Use Case**      | For dedicated nodes (e.g., GPU nodes) or special conditions. | For workload placement based on node characteristics (e.g., region). |
| **Configuration** | Taints applied to nodes, Tolerations added to Pods. | Node Affinity rules defined in Pod spec.        |
| **Effect**        | `NoSchedule`, `PreferNoSchedule`, or `NoExecute`.   | `RequiredDuringSchedulingIgnoredDuringExecution` or `PreferredDuringSchedulingIgnoredDuringExecution`. |

### Expected Results

- **`test-pod-blue`**: Will be in `Pending` state as it does not tolerate the `color=red` taint.
- **`test-pod-red`**: Will be in `Running` state as it tolerates the `color=red` taint.
- **`node-exporter`**: Will continue running on all nodes, including tainted nodes, as DaemonSets ignore taints by default.




