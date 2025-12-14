# Helm: ArgoCD Installation

## Add the Argo Helm Repository

```bash
helm repo add argo https://argoproj.github.io/argo-helm
```

## Preview the Chart (Template)

Generates manifests locally without installing:

```bash
helm template argocd argo/argo-cd --version <version> --skip-crds
```

### Save Template to File

Redirect to a specific file:

```bash
helm template argocd argo/argo-cd --version <version> --skip-crds > argocd.yaml
```

Or use `--output-dir` to write to a directory (preserves chart file structure):

```bash
helm template argocd argo/argo-cd --version <version> --skip-crds --output-dir ./manifests
```

## Install the Chart

```bash
helm install argocd argo/argo-cd --version <version> --skip-crds
```

## Key Flags

| Flag | Purpose |
|------|---------|
| `--version` | Specify chart version |
| `--skip-crds` | Exclude CRD manifests |

## Notes

- `helm template` only renders manifests — it does not install anything
- `helm install` tracks the release in Helm's state, enabling `upgrade`, `rollback`, and `uninstall`
- To install from a rendered template, pipe to kubectl: `helm template ... | kubectl apply -f -`
