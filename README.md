# arch-infra

GitOps source of truth for the k3s cluster on `aaron-desktop-arch`.

Argo CD watches this repo and reconciles cluster state to match.

## Layout

```
bootstrap/   # Argo CD app-of-apps root (only thing applied by hand)
infra/       # cluster-wide foundations (sealed-secrets, cert-manager, etc.)
platform/    # shared services (observability: loki, grafana, alloy)
apps/        # Argo CD Application CRs — central registry pointing at each app's own repo
```

## Stack

- **k3s** — single-node Kubernetes
- **Argo CD** — pull-based GitOps, watches this repo
- **GitHub Actions** — CI: build images, push to GHCR, bump tags here
- **GHCR** — container registry (`ghcr.io/autumnfallenwang/<app>`)
- **Sealed Secrets** — encrypted secrets safe to commit
- **Loki + Grafana + Alloy** — centralized logs

## Adding a new app

1. App lives in its own repo with `deploy/` folder containing Helm chart or manifests.
2. App's GHA workflow builds image → pushes to GHCR → bumps tag in this repo.
3. Add an Argo CD `Application` CR under `apps/<name>-application.yaml` pointing at the app repo's `deploy/` path.
4. Argo CD picks it up automatically (app-of-apps).
