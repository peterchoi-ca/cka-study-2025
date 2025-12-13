# Dead Cluster Troubleshooting Checklist
## Scenario: Unresponsive node, post-upgrade, external etcd

| Step | What to Check | Commands | What You're Looking For | Fix |
|------|---------------|----------|------------------------|-----|
| 1 | **etcd service status** | `systemctl status etcd` | Is the service running? | `systemctl start etcd && systemctl enable etcd` |
| 2 | **kubelet service status** | `systemctl status kubelet`<br>`journalctl -u kubelet -n 50` | Is kubelet running? Any errors connecting to API server? | `systemctl restart kubelet` |
| 3 | **Control plane manifests** | `ls -la /etc/kubernetes/manifests/`<br>`cat /etc/kubernetes/manifests/kube-apiserver.yaml` | Are files present and non-empty? | See Step 5 if empty |
| 4 | **Reconnaissance for config files** | `ls -la /etc/kubernetes/`<br>`ls -la /root/`<br>`find /etc/kubernetes -name "*.yaml" -type f`<br>`find /root -name "*.yaml" -type f` | Look for kubeadm-config.yaml or similar | Use found config in Step 5 |
| 5 | **Regenerate control plane** (if manifests empty) | `kubeadm init phase control-plane all --config=/path/to/kubeadm-config.yaml` | N/A | N/A |
| 6 | **Regenerate without config file** (if no config found) | `kubeadm init phase control-plane all \`<br>`--etcd-servers=https://<etcd-ip>:2379 \`<br>`--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt \`<br>`--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt \`<br>`--etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key` | N/A | N/A |
| 7 | **Verify cluster** | `kubectl get nodes`<br>`kubectl get pods -n kube-system` | Nodes ready, control plane pods running | Done! |

---

## Gathering External etcd Info (for Step 6)

| Info Needed | Commands |
|-------------|----------|
| etcd endpoint IP/port | `systemctl cat etcd`<br>`cat /etc/etcd/etcd.conf`<br>`cat /etc/default/etcd` |
| Certificate paths | `ls -la /etc/kubernetes/pki/`<br>`ls -la /etc/kubernetes/pki/etcd/` |
| Kubernetes version | `kubelet --version` |

---

## Quick Reference: Stacked vs External etcd

| Type | How etcd Runs | Config Location | Management |
|------|---------------|-----------------|------------|
| Stacked | Static pod | `/etc/kubernetes/manifests/etcd.yaml` | kubelet |
| External | systemd service | `/etc/etcd/` or systemd unit | `systemctl` |

---

## Exam Strategy

1. **Read the question carefully** — file paths are often provided
2. **60 seconds of recon first** — `ls` key directories before diving in
3. **Simplest fix first** — `systemctl start etcd` might be all you need
4. **Follow the chain** — etcd → kube-apiserver → kubectl
