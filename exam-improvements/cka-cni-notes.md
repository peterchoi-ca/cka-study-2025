# CNI Plugins & Network Policies

## Quick Comparison

| Feature | Calico | Flannel |
|---------|--------|---------|
| Network Policies | ✅ Yes (built-in) | ❌ No |
| Complexity | More features | Simpler |
| Use Case | Production, security-focused | Basic networking only |

## Key Exam Point

**If asked to install a CNI that supports NetworkPolicies → Choose Calico, not Flannel.**

## Installation (Tigera Operator Method)

### Step 1: Find the Cluster's PodCIDR

```bash
# Option A: Quick jsonpath (exam-friendly)
kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}'

# Option B: Per-node breakdown
kubectl get nodes -o custom-columns='NAME:.metadata.name,PODCIDR:.spec.podCIDR'

# Option C: Describe node
kubectl describe node <node-name> | grep PodCIDR

# Option D: kubeadm config
kubectl get cm kubeadm-config -n kube-system -o yaml | grep podSubnet

# Option E: Controller-manager manifest (requires SSH to control plane)
cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep cluster-cidr
```

**Exam tip:** Options A-C are fastest since they don't require SSH. Use jsonpath for quick retrieval.

### Step 2: Install the Tigera Operator
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/tigera-operator.yaml
```

### Step 3: Download and Edit custom-resources.yaml
```bash
# Download (same directory as tigera-operator.yaml)
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/custom-resources.yaml

# Edit the cidr field to match your cluster's PodCIDR
vim custom-resources.yaml
```

In `custom-resources.yaml`, find and update the `cidr` field:
```yaml
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 192.168.0.0/16    # ← Change this to match your cluster's PodCIDR
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
```

### Step 4: Apply the Custom Resources
```bash
kubectl create -f custom-resources.yaml
```

## Verification

```bash
# Check Calico pods are running
kubectl get pods -n calico-system

# Verify nodes are Ready
kubectl get nodes

# Check the Installation CR status
kubectl get installation default -o yaml
```

## Remember

- Flannel = Layer 3 networking only, no policy enforcement
- Calico = Full CNI with NetworkPolicy support out of the box
- Tigera operator uses `calico-system` namespace (not `kube-system`)
- `custom-resources.yaml` is in the same directory as `tigera-operator.yaml`
- The CIDR in Installation CRD must match the cluster's `--pod-network-cidr`
