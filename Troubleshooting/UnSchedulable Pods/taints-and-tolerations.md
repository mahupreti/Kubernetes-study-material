# Taints and Tolerations Troubleshooting Guide

## Problem Description

Taints allow nodes to repel pods, while tolerations allow pods to be scheduled on tainted nodes. When pods don't have the required tolerations, they cannot be scheduled on tainted nodes.

## Common Error Messages

```
Warning  FailedScheduling  <timestamp>  default-scheduler  0/3 nodes are available: 3 node(s) had taint {key: value}, that the pod didn't tolerate.
```

## Understanding Taints and Tolerations

### How Taints Work
- **Taints** are applied to nodes to repel pods
- **Tolerations** are applied to pods to tolerate specific taints
- A pod must tolerate ALL taints on a node to be scheduled there

### Taint Effects

#### NoSchedule
- New pods without tolerations will NOT be scheduled
- Existing pods remain running

#### PreferNoSchedule  
- Scheduler tries to avoid placing pods on the node
- Not a hard requirement

#### NoExecute
- New pods won't be scheduled AND existing pods will be evicted
- Most restrictive effect

## Diagnosis Steps

### Step 1: Check Node Taints
```bash
# Check all node taints
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints

# Check specific node taints
kubectl describe node <node-name> | grep Taints

# Check taints in JSON format
kubectl get node <node-name> -o jsonpath='{.spec.taints}'
```

### Step 2: Check Pod Tolerations
```bash
# Check pod tolerations
kubectl get pod <pod-name> -o yaml | grep -A10 tolerations

# Check pod in JSON format
kubectl get pod <pod-name> -o jsonpath='{.spec.tolerations}'
```

### Step 3: Match Taints with Tolerations
```bash
# List all tainted nodes
kubectl get nodes -o jsonpath='{range .items[?(@.spec.taints)]}{.metadata.name}{"\t"}{.spec.taints[*].key}{"\n"}{end}'
```

## Common Node Taints

### Built-in Kubernetes Taints
```bash
# Node not ready
node.kubernetes.io/not-ready:NoSchedule

# Node unreachable
node.kubernetes.io/unreachable:NoSchedule

# Node unschedulable
node.kubernetes.io/unschedulable:NoSchedule

# Memory pressure
node.kubernetes.io/memory-pressure:NoSchedule

# Disk pressure
node.kubernetes.io/disk-pressure:NoSchedule

# PID pressure
node.kubernetes.io/pid-pressure:NoSchedule

# Network unavailable
node.kubernetes.io/network-unavailable:NoSchedule
```

### Custom Taints Examples
```bash
# Dedicated GPU nodes
dedicated=gpu:NoSchedule

# Maintenance mode
maintenance=true:NoExecute

# High memory nodes
memory=high:PreferNoSchedule

# SSD storage nodes
storage=ssd:NoSchedule
```

## Solutions

### Solution 1: Add Tolerations to Pods

#### Basic Toleration (Exact Match)
```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu"
  effect: "NoSchedule"
```

#### Toleration with Exists Operator
```yaml
tolerations:
- key: "dedicated"
  operator: "Exists"
  effect: "NoSchedule"
```

#### Tolerate All Taints
```yaml
tolerations:
- operator: "Exists"
```

#### Tolerate Specific Key (Any Value)
```yaml
tolerations:
- key: "dedicated"
  operator: "Exists"
```

### Solution 2: Remove or Modify Node Taints

#### Remove Specific Taint
```bash
# Remove taint by key
kubectl taint node <node-name> <key>-

# Examples:
kubectl taint node worker-1 dedicated-
kubectl taint node worker-2 maintenance-
kubectl taint node worker-3 storage=ssd:NoSchedule-
```

#### Add Taint to Node
```bash
# Add taint with effect
kubectl taint node <node-name> <key>=<value>:<effect>

# Examples:
kubectl taint node worker-1 dedicated=gpu:NoSchedule
kubectl taint node worker-2 maintenance=true:NoExecute
kubectl taint node worker-3 storage=ssd:PreferNoSchedule
```

#### Modify Taint Effect
```bash
# Remove old taint and add new one
kubectl taint node <node-name> dedicated=gpu:NoSchedule-
kubectl taint node <node-name> dedicated=gpu:PreferNoSchedule
```

### Solution 3: Emergency Fixes

#### Remove All Taints from Node
```bash
kubectl patch node <node-name> -p '{"spec":{"taints":[]}}'
```

#### Force Schedule Pod (Bypass Scheduler)
```bash
kubectl patch pod <pod-name> -p '{"spec":{"nodeName":"<node-name>"}}'
```

## Example YAML Files

### Pod with Specific Tolerations
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-tolerations
spec:
  containers:
  - name: nginx
    image: nginx:latest
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
  - key: "maintenance"
    operator: "Exists"
    effect: "PreferNoSchedule"
  - key: "storage"
    operator: "Equal"
    value: "ssd"
    effect: "NoSchedule"
    tolerationSeconds: 300
```

### Pod Tolerating All Taints
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-tolerate-all
spec:
  containers:
  - name: nginx
    image: nginx:latest
  tolerations:
  - operator: "Exists"
```

### Deployment with Tolerations
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gpu-app
  template:
    metadata:
      labels:
        app: gpu-app
    spec:
      containers:
      - name: gpu-app
        image: nvidia/cuda:latest
      tolerations:
      - key: "nvidia.com/gpu"
        operator: "Exists"
        effect: "NoSchedule"
      - key: "dedicated"
        operator: "Equal"
        value: "gpu"
        effect: "NoSchedule"
```

### DaemonSet with Node Tolerations
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-agent
spec:
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      containers:
      - name: agent
        image: monitoring:latest
      tolerations:
      - operator: "Exists"  # Tolerate all taints
```

## Toleration Operators

### Equal
Must match key, value, and effect exactly:
```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "gpu"
  effect: "NoSchedule"
```

### Exists
Matches if key exists (ignores value):
```yaml
tolerations:
- key: "dedicated"
  operator: "Exists"
  effect: "NoSchedule"
```

## Advanced Scenarios

### Time-Limited Tolerations
```yaml
tolerations:
- key: "maintenance"
  operator: "Equal"
  value: "true"
  effect: "NoExecute"
  tolerationSeconds: 3600  # Tolerate for 1 hour
```

### Multiple Toleration Strategies
```yaml
tolerations:
# Exact match for production
- key: "environment"
  operator: "Equal"
  value: "production"
  effect: "NoSchedule"
  
# Exists match for any GPU
- key: "gpu"
  operator: "Exists"
  effect: "NoSchedule"
  
# Tolerate maintenance for 30 minutes
- key: "maintenance"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 1800
```

## Testing Commands

### Test Taint and Toleration Setup
```bash
# Apply taint to test node
kubectl taint node test-node dedicated=gpu:NoSchedule

# Apply test pod with toleration
kubectl apply -f pod-with-tolerations.yaml

# Check if pod scheduled
kubectl get pod <pod-name> -o wide

# Clean up
kubectl taint node test-node dedicated-
kubectl delete pod <pod-name>
```

### Bulk Operations
```bash
# Taint multiple nodes
kubectl taint nodes worker-1 worker-2 worker-3 dedicated=gpu:NoSchedule

# Remove taint from multiple nodes
kubectl taint nodes worker-1 worker-2 worker-3 dedicated-
```

## Common Use Cases

### 1. Dedicated GPU Nodes
```bash
# Taint GPU nodes
kubectl taint node gpu-node-1 dedicated=gpu:NoSchedule

# Pod toleration
tolerations:
- key: "dedicated"
  value: "gpu"
  effect: "NoSchedule"
```

### 2. Master Node Isolation
```bash
# Master nodes usually have this taint
kubectl taint node master-node node-role.kubernetes.io/master:NoSchedule

# System pods tolerate this
tolerations:
- key: "node-role.kubernetes.io/master"
  effect: "NoSchedule"
```

### 3. Maintenance Mode
```bash
# Put node in maintenance
kubectl taint node worker-1 maintenance=true:NoExecute

# Pods with toleration can stay
tolerations:
- key: "maintenance"
  value: "true"
  effect: "NoExecute"
  tolerationSeconds: 3600
```

## Best Practices

1. **Use meaningful taint keys**: `dedicated`, `maintenance`, `storage-type`
2. **Document your tainting strategy** clearly
3. **Use PreferNoSchedule** for soft constraints
4. **Test tolerations** before applying to production
5. **Monitor tainted nodes** regularly
6. **Use tolerationSeconds** for temporary taints
7. **Combine with nodeSelector/nodeAffinity** for precise placement
8. **Plan for emergency scenarios** (remove all taints)

## Troubleshooting Checklist

- Check node taints: `kubectl describe node <node-name>`
- Check pod tolerations: `kubectl get pod <pod-name> -o yaml`
- Verify taint key, value, and effect match exactly
- Test with simplified toleration (operator: "Exists")
- Check for multiple taints on same node
- Verify tolerationSeconds if using NoExecute
- Consider using emergency fixes for urgent situations