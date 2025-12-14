# CRI-Dockerd Study Notes

## Overview

Since Kubernetes 1.24, Docker Engine is no longer a built-in container runtime. **cri-dockerd** acts as a shim/bridge that allows Kubernetes to communicate with Docker Engine via the Container Runtime Interface (CRI).

## Common Socket Paths

| Runtime | Socket Path |
|---------|-------------|
| cri-dockerd | `unix:///var/run/cri-dockerd.sock` |
| containerd | `unix:///var/run/containerd/containerd.sock` |

---

## Installation

```bash
# Download from provided GitHub URL
wget <provided-github-url>

# Install (if .deb package)
sudo dpkg -i cri-dockerd_*.deb
```

## Enable and Start the Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable cri-dockerd
sudo systemctl start cri-dockerd

# Verify
sudo systemctl status cri-dockerd
```

---

## Using cri-dockerd with kubeadm

### Joining a Node to a Cluster

```bash
kubeadm join <control-plane-endpoint>:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --cri-socket unix:///var/run/cri-dockerd.sock
```

### Initializing a New Cluster

```bash
kubeadm init \
  --cri-socket unix:///var/run/cri-dockerd.sock \
  --pod-network-cidr=10.244.0.0/16
```

---

## Configuring an Existing Kubelet

Edit the kubelet configuration:

```bash
sudo vi /var/lib/kubelet/kubeadm-flags.env
```

Add or modify:

```
KUBELET_KUBEADM_ARGS="--container-runtime-endpoint=unix:///var/run/cri-dockerd.sock"
```

Restart kubelet:

```bash
sudo systemctl restart kubelet
```

---

## Verification Commands

```bash
# Check runtime version on a node
kubectl get node <node-name> -o jsonpath='{.status.nodeInfo.containerRuntimeVersion}'

# View runtime in node list
kubectl get nodes -o wide
# Look at CONTAINER-RUNTIME column

# Describe node for runtime info
kubectl describe node <node-name> | grep -i runtime
```

---

## Key Points to Remember

- `--cri-socket` → used with **kubeadm** commands (init, join)
- `--container-runtime-endpoint` → used in **kubelet** configuration
- Both flags point to the same socket path
- Always run `systemctl daemon-reload` after installing new services
- Use `systemctl enable` to ensure service starts on boot
