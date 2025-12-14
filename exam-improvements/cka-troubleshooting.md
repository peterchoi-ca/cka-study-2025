# CKA Troubleshooting Reference

## Static Pod Manifests

Location: `/etc/kubernetes/manifests/`

| File | When to Check |
|------|---------------|
| `etcd.yaml` | Cluster state issues, API server can't start, "connection refused" errors, data directory problems |
| `kube-apiserver.yaml` | Authentication/authorization failures, can't connect to cluster, certificate issues, RBAC not working |
| `kube-controller-manager.yaml` | Deployments not scaling, ReplicaSets not creating pods, services not getting endpoints |
| `kube-scheduler.yaml` | Pods stuck in Pending (but node has capacity), scheduling decisions not happening |

## Critical File Paths

| Path | Purpose |
|------|---------|
| `/etc/kubernetes/pki/` | All certificates and keys — check when TLS/cert errors appear |
| `/etc/kubernetes/kubelet.conf` | Kubelet's kubeconfig for talking to API server |
| `/etc/kubernetes/admin.conf` | Admin kubeconfig (often symlinked to `~/.kube/config`) |
| `/var/lib/kubelet/config.yaml` | Kubelet configuration (staticPodPath, clusterDNS, etc.) |
| `/var/lib/etcd/` | etcd data directory — check permissions, disk space |

## Log Commands

```bash
# Kubelet logs (essential for static pod issues)
journalctl -u kubelet -f

# Control plane pod logs
kubectl logs -n kube-system kube-apiserver-controlplane
kubectl logs -n kube-system etcd-controlplane
kubectl logs -n kube-system kube-scheduler-controlplane
kubectl logs -n kube-system kube-controller-manager-controlplane

# Container runtime level (when kubectl doesn't work)
crictl ps -a
crictl logs <container-id>
```

## Troubleshooting by Symptom

| Symptom | First Place to Look |
|---------|---------------------|
| `kubectl` commands hang or timeout | `kube-apiserver.yaml`, then `etcd.yaml` |
| Pods stuck Pending, no events | `kube-scheduler.yaml` |
| Deployments stuck, not creating pods | `kube-controller-manager.yaml` |
| Node NotReady | `journalctl -u kubelet`, kubelet config |
| Static pod not starting | `journalctl -u kubelet` (not kubectl logs) |
| Certificate errors | `/etc/kubernetes/pki/`, check expiry |

## Certificate Commands

```bash
# Check certificate details and expiry
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout

# Check CSR
openssl req -in myuser.csr -text -noout

# Generate private key
openssl genrsa -out myuser.key 2048

# Generate CSR
openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser/O=developers"

# Base64 encode CSR for K8s CertificateSigningRequest
cat myuser.csr | base64 | tr -d '\n'
```

## Kubernetes CSR Workflow

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
```

```bash
# Approve the CSR
kubectl certificate approve myuser

# Get the signed certificate
kubectl get csr myuser -o jsonpath='{.status.certificate}' | base64 -d > myuser.crt
```

## Tips

- Static pod manifests are auto-reloaded by kubelet — no restart needed after edits
- If static pod doesn't cycle after fix, check `journalctl -u kubelet` for YAML parse errors
- Always check `kubectl get events` for recent cluster events
- Use `kubectl describe` liberally — events section at bottom is gold
- For node issues, SSH to the node and check kubelet directly
