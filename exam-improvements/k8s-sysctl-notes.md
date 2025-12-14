# Persistent Sysctl Settings for Kubernetes

## Overview

Kubernetes requires specific kernel parameters for networking (iptables) to function. These must persist across reboots.

## Required Parameters

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `net.bridge.bridge-nf-call-iptables` | 1 | Allow iptables to process bridged IPv4 traffic |
| `net.bridge.bridge-nf-call-ip6tables` | 1 | Allow iptables to process bridged IPv6 traffic |
| `net.ipv4.ip_forward` | 1 | Enable IP forwarding between interfaces |

## Configure Sysctl Parameters (Persistent)

```bash
# Create persistent config file
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply immediately without reboot
sudo sysctl --system

# Verify settings
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.bridge.bridge-nf-call-ip6tables
sysctl net.ipv4.ip_forward
```

## Key Locations

| Path | Purpose |
|------|---------|
| `/etc/sysctl.conf` | Main sysctl config (legacy) |
| `/etc/sysctl.d/*.conf` | Drop-in config files (preferred) |
| `/etc/modules-load.d/*.conf` | Kernel modules to load at boot |

## Quick Reference Commands

```bash
# Apply all sysctl configs from standard locations
sudo sysctl --system

# Apply specific file
sudo sysctl -p /etc/sysctl.d/k8s.conf

# View current value of a parameter
sysctl <parameter.name>

# Set temporarily (non-persistent)
sudo sysctl -w net.ipv4.ip_forward=1
```

## Exam Tips

- Use `/etc/sysctl.d/` for drop-in files (modern approach)
- Always run `sysctl --system` after creating config files
- Don't forget `br_netfilter` module — bridge settings won't work without it
- Verify your changes with `sysctl <param>` after applying
