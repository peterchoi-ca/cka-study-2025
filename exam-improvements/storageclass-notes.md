# StorageClass - CKA Study Notes

## Overview
A StorageClass provides a way to describe different "classes" of storage available in a cluster. It enables dynamic provisioning of PersistentVolumes.

## Creating a StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage-class
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

## Making a StorageClass Default

### Method 1: Include annotation in YAML
```yaml
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
```

### Method 2: Patch existing StorageClass
```bash
kubectl patch storageclass <name> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Removing Default from Existing StorageClass
If another StorageClass is already default, remove it first to avoid conflicts:
```bash
kubectl patch storageclass <old-default> -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```

## Common Provisioners
| Provisioner | Use Case |
|-------------|----------|
| `kubernetes.io/no-provisioner` | Local/static volumes |
| `kubernetes.io/aws-ebs` | AWS EBS |
| `kubernetes.io/gce-pd` | GCP Persistent Disk |
| `kubernetes.io/azure-disk` | Azure Disk |

## Volume Binding Modes
| Mode | Behavior |
|------|----------|
| `Immediate` | PV bound as soon as PVC is created |
| `WaitForFirstConsumer` | PV bound when Pod using PVC is scheduled |

## Reclaim Policies
| Policy | Behavior |
|--------|----------|
| `Retain` | Keep PV after PVC deletion (manual cleanup) |
| `Delete` | Delete PV when PVC is deleted |

## Verification Commands
```bash
# List all StorageClasses (default marked with "(default)")
kubectl get storageclass
kubectl get sc

# Describe a StorageClass
kubectl describe sc <name>
```

## Quick Reference
- **No imperative command** exists for creating StorageClass - must use YAML
- **Default annotation**: `storageclass.kubernetes.io/is-default-class: "true"`
- **Only one default** should exist per cluster
- PVCs without a `storageClassName` use the default StorageClass
