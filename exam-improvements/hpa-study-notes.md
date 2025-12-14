# Horizontal Pod Autoscaler (HPA) - CKA Study Notes

## Overview

HPA automatically scales pod replicas in a Deployment, ReplicaSet, or StatefulSet based on observed metrics (typically CPU/memory utilization). It runs as a control loop that periodically queries metrics and adjusts replicas.

**Scaling Formula:**
```
desiredReplicas = ceil(currentReplicas × (currentMetricValue / targetMetricValue))
```

---

## Key Components

| Component | Purpose |
|-----------|---------|
| **Metrics Server** | Collects resource metrics from kubelets; required for HPA to function |
| **scaleTargetRef** | Reference to the workload being scaled (Deployment, ReplicaSet, StatefulSet) |
| **minReplicas** | Lower bound for scaling |
| **maxReplicas** | Upper bound for scaling |
| **metrics** | Target metrics (CPU, memory, custom, external) |

### Supported Metric Types
- **Resource metrics** — CPU, memory (most common on CKA)
- **Custom metrics** — Application-specific metrics
- **External metrics** — Metrics from outside the cluster

---

## Essential Commands

```bash
# Create HPA imperatively (exam-friendly)
kubectl autoscale deployment <name> --min=<n> --max=<n> --cpu-percent=<n>

# Example
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=50

# Generate YAML without creating
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=50 --dry-run=client -o yaml

# Check HPA status
kubectl get hpa
kubectl get hpa --watch

# Detailed information
kubectl describe hpa <name>

# Edit existing HPA
kubectl edit hpa <name>

# Patch HPA
kubectl patch hpa <name> --patch '{"spec":{"minReplicas":5,"maxReplicas":15}}'

# Delete HPA
kubectl delete hpa <name>
```

---

## Basic HPA YAML (autoscaling/v2)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

### Multi-Metric HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: multi-metric-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70
```

> **Note:** With multiple metrics, HPA calculates desired replicas for each and uses the **highest** value.

---

## CKA Exam Scenarios

### Scenario 1: Create HPA for Existing Deployment

**Task:** Create HPA for deployment `web-app` in namespace `production`, scaling between 3-8 replicas at 60% CPU.

**Solution:**
```bash
kubectl autoscale deployment web-app -n production --min=3 --max=8 --cpu-percent=60
```

**Verify:**
```bash
kubectl get hpa -n production
```

---

### Scenario 2: Troubleshoot `<unknown>` Metrics

**Symptom:**
```
NAME         REFERENCE              TARGETS         MINPODS   MAXPODS   REPLICAS
api-server   Deployment/api-server  <unknown>/50%   2         10        2
```

**Diagnostic Steps:**

```bash
# Check HPA details
kubectl describe hpa api-server

# Verify metrics-server is running
kubectl get pods -n kube-system | grep metrics-server
kubectl top nodes
kubectl top pods
```

**Common Causes & Fixes:**

| Cause | Fix |
|-------|-----|
| Metrics Server not running | Install/restart metrics-server |
| **No resource requests on pods** | Add `resources.requests` to pod spec |

**Fix missing resource requests:**
```bash
kubectl edit deployment api-server
```

Add under container spec:
```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
```

> **Critical:** Pods MUST have resource requests defined for CPU/memory-based HPA to calculate utilization percentages.

---

### Scenario 3: Modify Existing HPA

**Task:** Update HPA `frontend-hpa` to min=5, max=15 replicas.

**Option A - Edit directly:**
```bash
kubectl edit hpa frontend-hpa
```

**Option B - Patch:**
```bash
kubectl patch hpa frontend-hpa --patch '{"spec":{"minReplicas":5,"maxReplicas":15}}'
```

**Option C - Export, modify, replace:**
```bash
kubectl get hpa frontend-hpa -o yaml > hpa.yaml
# Edit the file
kubectl replace -f hpa.yaml
```

---

### Scenario 4: Verify Scaling Behavior

**Commands:**
```bash
# Quick status
kubectl get hpa order-service

# Detailed with events
kubectl describe hpa order-service
```

**Key Conditions in `describe` output:**

| Condition | Meaning |
|-----------|---------|
| **AbleToScale** | Can HPA make scaling decisions? |
| **ScalingActive** | Are metrics being collected successfully? |
| **ScalingLimited** | Is scaling constrained by min/max bounds? |

**Simulate load for testing:**
```bash
# Run load generator
kubectl run load-gen --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://service-name; done"

# Watch HPA respond
kubectl get hpa --watch
```

---

## Dependency Chain

```
metrics-server (running)
        ↓
pods have resource requests defined
        ↓
HPA can calculate utilization %
        ↓
HPA scales replicas
```

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Create HPA | `kubectl autoscale deployment <name> --min=2 --max=10 --cpu-percent=50` |
| List HPAs | `kubectl get hpa` |
| Describe HPA | `kubectl describe hpa <name>` |
| Edit HPA | `kubectl edit hpa <name>` |
| Watch HPA | `kubectl get hpa --watch` |
| Check metrics server | `kubectl top pods` |
| Generate YAML | `kubectl autoscale ... --dry-run=client -o yaml` |

---

## Exam Tips

1. **Use imperative commands** — `kubectl autoscale` is faster and less error-prone than writing YAML
2. **Bookmark the HPA docs page** — YAML examples are readily available
3. **Always check first** — Run `kubectl get hpa` to understand current state before changes
4. **Remember the dependency** — metrics-server → resource requests → HPA functionality
5. **Use `--watch`** — Observe scaling in real-time when verifying behavior
6. **`<unknown>` usually means** — Missing resource requests on pods (most common exam gotcha)
