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

### Command Syntax Breakdown

```
helm install <release-name> <repo-name>/<chart-name>
```

| Component | Example | Description |
|-----------|---------|-------------|
| Release name | `argocd` | Your chosen name for this installation |
| Repo name | `argo` | From `helm repo add argo https://...` |
| Chart name | `argo-cd` | The chart within that repo |

Multiple releases from the same chart are possible by changing the release name:

```bash
helm install argocd-dev argo/argo-cd
helm install argocd-prod argo/argo-cd
```

### What Helm Can Install

Helm can only install *charts*, not rendered YAML files:

| Source | Install Command |
|--------|-----------------|
| Chart from repo | `helm install argocd argo/argo-cd` |
| Local chart directory | `helm install argocd ./argo-cd` |
| Packaged chart (.tgz) | `helm install argocd argo-cd-1.0.0.tgz` |
| Rendered YAML file | `kubectl apply -f argocd.yaml` (not Helm) |

## Key Flags

| Flag | Purpose |
|------|---------|
| `--version` | Specify chart version |
| `--skip-crds` | Exclude CRD manifests |

## Notes

- `helm template` only renders manifests — it does not install anything
- `helm install` tracks the release in Helm's state, enabling `upgrade`, `rollback`, and `uninstall`
- To install from a rendered template, pipe to kubectl: `helm template ... | kubectl apply -f -`
