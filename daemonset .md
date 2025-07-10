
# DaemonSet in Kubernetes

A **DaemonSet** ensures that a specific pod runs on every (or specific) node in a Kubernetes cluster. It's essential for running infrastructure services like log collectors, monitoring agents, or node-specific utilities.

---

## ✅ Key Features of DaemonSets

### 1. One Pod per Node

* Ensures that exactly one pod is scheduled per node (unless configured otherwise).
* If a new node joins the cluster, a pod is automatically scheduled on it.

### 2. Automatic Pod Management

* Handles pod lifecycle automatically (adds to new nodes, removes from deleted nodes).

### 3. Selective Node Deployment

* Can be restricted to specific nodes using:

  * `nodeSelector`
  * `nodeAffinity`
  * `Taints and Tolerations`

---

## 📌 Common Use Cases

| Use Case   | Description                                                       |
| ---------- | ----------------------------------------------------------------- |
| Monitoring | Deploy node-level monitoring agents like Prometheus node-exporter |
| Logging    | Run log shippers like Fluentd or Filebeat on each node            |
| Networking | Install network tools like Cilium, Calico, or Envoy               |
| Security   | Run vulnerability scanners or antivirus agents                    |

---

## 📘 Example 1: Basic DaemonSet (Runs only on worker nodes)

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
spec:
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
        image: nginx
        ports:
        - containerPort: 80
```

**Note:**
By default, pods will **not be scheduled on master/control-plane nodes** due to taints.

### Check taints on nodes:

```bash
kubectl describe node <node-name>
```

---

## 📘 Example 2: Allow DaemonSet Pods on Control Plane Node

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

---

## ⚠️ Common Issues with DaemonSets

| Issue                           | Description                      | Troubleshooting                       |
| ------------------------------- | -------------------------------- | ------------------------------------- |
| Pods not running on master      | Node is tainted                  | Add toleration in pod spec            |
| Pods not deployed to some nodes | Selector/affinity issue          | Check node labels and selectors       |
| CrashLoopBackOff                | App inside pod is crashing       | Check logs, probe configs, env vars   |
| Scheduling fails                | Resource constraints or affinity | Describe pod, check node availability |
| Resource limits issues          | Not enough CPU/memory on node    | Adjust requests/limits                |

---

## 🛠️ Troubleshooting Steps

1. **Check Pod Events:**

```bash
kubectl describe pod <pod-name> -n <namespace>
```

2. **View Logs:**

```bash
kubectl logs <pod-name> -n <namespace>
```

3. **Verify Node Labels/Taints:**

```bash
kubectl get nodes --show-labels
kubectl describe node <node-name>
```

4. **Check DaemonSet Status:**

```bash
kubectl get daemonset -n <namespace>
kubectl describe daemonset <name> -n <namespace>
```

---

## 🎯 Real-Time Scenario Based Questions and Answers

### ✅ Q1: You deployed a DaemonSet, but pods are not running on all nodes. What could be wrong?

**A:**
Check node taints using `kubectl describe node`. Control plane nodes have a taint like `node-role.kubernetes.io/control-plane:NoSchedule`. Ensure your DaemonSet pods tolerate this. Also verify nodeSelector or affinity rules.

---

### ✅ Q2: A DaemonSet pod is in CrashLoopBackOff state. How would you troubleshoot?

**A:**
Check logs with `kubectl logs --previous`, then inspect the pod definition for missing env variables, misconfigured volume mounts, or resource constraints. Adjust probes or limits if necessary.

---

### ✅ Q3: Can you explain a real-time use of DaemonSet in your project?

**A:**
We used a DaemonSet to deploy Filebeat on all cluster nodes to ship logs to Elasticsearch. It was configured to tolerate master taints so logs from control-plane were also captured during debugging.

---

### ✅ Q4: How can you restrict DaemonSet pods to only a subset of nodes?

**A:**
Label specific nodes and use `nodeSelector`:

```bash
kubectl label node node-1 log-agent=true
```

Update DaemonSet:

```yaml
nodeSelector:
  log-agent: "true"
```

---

### ✅ Q5: How does DaemonSet behave differently from a Deployment?

**A:**
A **Deployment** ensures a specified number of replicas across the cluster, while a **DaemonSet** ensures **one pod per node**. DaemonSet is for node-specific workloads, unlike stateless apps for Deployment.

---

Let me know if you need a version with diagrams or YAML best practices added!
