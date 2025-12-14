# CNI Plugins & Network Policies

## Quick Comparison

| Feature | Calico | Flannel |
|---------|--------|---------|
| Network Policies | ✅ Yes (built-in) | ❌ No |
| Complexity | More features | Simpler |
| Use Case | Production, security-focused | Basic networking only |

## Key Exam Point

**If asked to install a CNI that supports NetworkPolicies → Choose Calico, not Flannel.**

Calico enables NetworkPolicy support automatically upon installation—no extra configuration required.

## Installation

### Option 1: Direct Apply (if CIDR matches default)
```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml
```

### Option 2: Download, Edit, Apply (if custom CIDR needed)
```bash
# Download the manifest
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

# Edit CALICO_IPV4POOL_CIDR to match your cluster's --pod-network-cidr
vim calico.yaml

# Apply
kubectl apply -f calico.yaml
```

## Verification

```bash
# Check Calico pods are running
kubectl get pods -n kube-system | grep calico

# Verify nodes are Ready (CNI installed)
kubectl get nodes
```

## Remember

- Flannel = Layer 3 networking only, no policy enforcement
- Calico = Full CNI with NetworkPolicy support out of the box
- Installing Calico IS enabling NetworkPolicy support
