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

## Services

What runs on the cluster. The apps' hostnames resolve through the router's DNS, one name at a time, to Traefik on
192.168.1.163. The two gateways have no login, so they have no Ingress. On arch, `/etc/hosts` points their full
names at their pinned Service IPs. llmgw is for pods and arch; mcpgw is for the home apps.

| Service | What it's for | URL | Repo |
|---|---|---|---|
| llmgw | **Models.** One place to reach LLMs: routing, logins, limits, usage | http://llmgw.llmgw.svc.cluster.local (arch and pods only) | [llm-gateway](https://github.com/autumnfallenwang/llm-gateway) |
| mcpgw | **Tools.** Web search and page reading over MCP, for the home apps | http://mcpgw.mcpgw.svc.cluster.local (home apps only) | [mcpgw](https://github.com/autumnfallenwang/mcpgw) |
| homestorm | Brainstorm with several models at once, with you as chair | http://homestorm.arch.internal | [homestorm](https://github.com/autumnfallenwang/homestorm) |
| homecal | Smart family calendar | http://homecal.arch.internal | [homecal](https://github.com/autumnfallenwang/homecal) |
| homenews | Personal AI news intelligence | http://homenews.arch.internal | [homenews](https://github.com/autumnfallenwang/homenews) |
| homework | School parent companion, moved from TeacherEase Parent Companion | http://homework.arch.internal | [homework](https://github.com/autumnfallenwang/homework) |
| homeparentcontrol | Parental control for a child's Mac | http://homeparentcontrol.arch.internal | [homeparentcontrol](https://github.com/autumnfallenwang/homeparentcontrol) |
| Argo CD | Syncs the cluster to this repo | http://argocd.arch.internal | this repo |
| Grafana, Loki | Logs | http://grafana.arch.internal | this repo |

**The split:** models live in llmgw, tools live in mcpgw, and when to use them lives in each app.

## Adding a new app

1. App lives in its own repo with `deploy/` folder containing Helm chart or manifests.
2. App's GHA workflow builds image → pushes to GHCR → bumps tag in this repo.
3. Add an Argo CD `Application` CR under `apps/<name>-application.yaml` pointing at the app repo's `deploy/` path.
4. Argo CD picks it up automatically (app-of-apps).
