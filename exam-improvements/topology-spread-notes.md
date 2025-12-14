# Pod Topology Spread Constraints

## Purpose
Distribute pods evenly across nodes, zones, or other topology domains to improve availability and resource utilization.

## When to Use
- Ensure high availability by spreading replicas across failure domains
- Balance load across nodes
- Exam scenario: "distribute pods evenly" or "spread pods across nodes"

---

## Configuration

Add under `spec.template.spec` in a Deployment:

```yaml
topologySpreadConstraints:
- maxSkew: 1
  topologyKey: kubernetes.io/hostname
  whenUnsatisfiable: DoNotSchedule
  labelSelector:
    matchLabels:
      app: my-app
```

---

## Key Fields

| Field | Description |
|-------|-------------|
| `maxSkew` | Max difference in pod count between topology domains. Use `1` for most even distribution. |
| `topologyKey` | Node label to spread across. Common values below. |
| `whenUnsatisfiable` | `DoNotSchedule` (hard) or `ScheduleAnyway` (soft preference) |
| `labelSelector` | Must match the pods you want to spread (usually same as deployment's selector) |

### Common topologyKey Values
- `kubernetes.io/hostname` — spread across nodes
- `topology.kubernetes.io/zone` — spread across zones
- `topology.kubernetes.io/region` — spread across regions

---

## Exam Procedure

**If existing pods need redistribution:**

```bash
# 1. Scale down to zero
kubectl scale deployment <name> --replicas=0

# 2. Edit deployment to add topologySpreadConstraints
kubectl edit deployment <name>

# 3. Scale back up
kubectl scale deployment <name> --replicas=<count>
```

> **Why scale down?** Topology constraints only apply to newly scheduled pods. Existing pods won't move automatically.

---

## Quick Reference

```yaml
# Minimal example - spread across nodes
spec:
  template:
    spec:
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: kubernetes.io/hostname
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: my-app
      containers:
      - name: my-container
        image: nginx
```

---

## Related Concepts
- **Pod Anti-Affinity**: Similar goal but uses different mechanism (`affinity.podAntiAffinity`)
- **Node Affinity**: Controls which nodes pods can schedule on (not distribution)
