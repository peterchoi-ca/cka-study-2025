# PriorityClass Study Notes (CKA)

## What is PriorityClass?

PriorityClass determines **pod scheduling priority** and **preemption behavior** in a resource-constrained cluster.

- **Higher value = Higher priority**
- Range: approximately -2 billion to +1 billion
- Pods with higher priority get scheduled first
- Lower priority pods can be **preempted** (evicted) to make room for higher priority pods

---

## Key Fields

| Field | Purpose |
|-------|---------|
| `value` | Integer priority (higher = more important) |
| `globalDefault` | If `true`, pods without a `priorityClassName` inherit this |
| `preemptionPolicy` | `PreemptLowerPriority` (default) or `Never` |
| `description` | Human-readable explanation |

---

## Common Commands

### List all PriorityClasses
```bash
kubectl get priorityclass
```

### Create imperatively
```bash
kubectl create priorityclass <name> --value=<int> --description="<text>"
```

### Dry-run to generate YAML
```bash
kubectl create priorityclass my-priority --value=1999999 --dry-run=client -o yaml
```

---

## YAML Manifest

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "High priority workloads"
```

---

## Assigning to a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: nginx
```

---

## System PriorityClasses (Do Not Modify)

| Name | Value |
|------|-------|
| `system-node-critical` | 2000001000 |
| `system-cluster-critical` | 2000000000 |

These are reserved for critical system components (kube-proxy, CoreDNS, etc.).

---

## Exam Tips

1. **Imperative creation is faster** — use `kubectl create priorityclass`
2. **Remember**: bigger number = higher priority
3. **Read carefully**: "one less than highest" means subtract 1 from the max value
4. **Don't touch** system PriorityClasses unless explicitly asked
5. **globalDefault**: only one PriorityClass should have this set to `true`

---

## Quick Reference

```bash
# Get all priority classes with values
kubectl get priorityclass

# Create new priority class
kubectl create priorityclass almost-high --value=1999999

# Check which priority class a pod uses
kubectl get pod <name> -o jsonpath='{.spec.priorityClassName}'
```
