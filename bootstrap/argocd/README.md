# Argo CD bootstrap

The only manifests applied by hand. After this, Argo CD takes over and manages everything (including itself) from this repo.

## Files

- `namespace.yaml` — `argocd` namespace
- `install.yaml` — vendored from `https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml` (snapshot for reproducibility)
- `kustomization.yaml` — combines them

## Apply

```sh
kubectl apply -k bootstrap/argocd/ --server-side --force-conflicts
```

`--server-side` is required: Argo CD's CRDs exceed the 262144-byte annotation
limit used by client-side apply.

## Upgrade Argo CD

```sh
curl -sfL https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml \
  -o bootstrap/argocd/install.yaml
git diff bootstrap/argocd/install.yaml   # review changes
git commit -am "bump argocd to <version>"
git push
# Argo CD self-syncs (once root-app is configured to manage this folder)
```
