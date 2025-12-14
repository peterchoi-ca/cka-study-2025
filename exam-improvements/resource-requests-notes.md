# Pod Resource Requests & Scheduling

## Scenario
Pods are **Pending** because resource requests exceed available node capacity. You need to check the node's resources and adjust container requests so all replicas can schedule.

---

## Step-by-Step Procedure

### 1. Identify the Problem

```bash
# Check why pods are pending
kubectl get pods
kubectl describe pod <pending-pod-name>
# Look for: "Insufficient cpu" or "Insufficient memory"
```

### 2. Check Node's Available Resources

```bash
kubectl describe node <node-name>
```

Look for two sections:

```
Capacity:
  cpu:     4
  memory:  8Gi

Allocatable:
  cpu:     3800m
  memory:  7600Mi

Allocated resources:
  (Total requests already in use)
```

> **Allocatable** = what's available for pods (Capacity minus system reserved)

### 3. Calculate Resource Requests

**Formula:**
```
Per-pod request = (Allocatable - buffer) / number of replicas
```

**Example calculation:**
- Node Allocatable: `cpu: 3800m`, `memory: 7600Mi`
- Reserve ~20% buffer for system/daemonsets: `cpu: ~760m`, `memory: ~1500Mi`
- Usable: `cpu: ~3000m`, `memory: ~6000Mi`
- 3 replicas: `cpu: 1000m`, `memory: 2000Mi` per pod

> **Tip:** Be conservative. Round down to leave headroom.

### 4. Scale Down Deployment

```bash
kubectl scale deployment <n> --replicas=0
```

### 5. Edit Deployment Resources

```bash
kubectl edit deployment <n>
```

Update the container's resources:

```yaml
spec:
  template:
    spec:
      containers:
      - name: my-container
        resources:
          requests:
            cpu: "1000m"
            memory: "2000Mi"
          limits:
            cpu: "1000m"
            memory: "2000Mi"
```

### 6. Scale Back Up

```bash
kubectl scale deployment <n> --replicas=3
```

### 7. Verify

```bash
kubectl get pods
# All pods should be Running, none Pending
```

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `kubectl describe node <n>` | Check Allocatable CPU/memory |
| `kubectl describe pod <p>` | See why pod is Pending |
| `kubectl scale deploy <d> --replicas=0` | Scale down |
| `kubectl edit deploy <d>` | Modify resource requests |

---

## Key Concepts

| Term | Meaning |
|------|---------|
| **Capacity** | Total node resources (hardware) |
| **Allocatable** | Resources available for pods (after system reservation) |
| **Requests** | Guaranteed resources; used by scheduler for placement |
| **Limits** | Maximum resources a container can use |

---

## Resource Units

| CPU | Memory |
|-----|--------|
| `1` = 1 vCPU/core | `1Gi` = 1 gibibyte |
| `1000m` = 1 CPU | `1G` = 1 gigabyte |
| `500m` = 0.5 CPU | `1Mi` = 1 mebibyte |

---

## Common Mistakes
- Forgetting to leave buffer for DaemonSets and system pods
- Setting requests too high (pods won't schedule)
- Confusing Capacity vs Allocatable
- Not scaling down before editing (existing pods keep old values)
