# gitops.teupadel.com

ArgoCD/Helm deployment repo for **teupadel.com** — AI-powered padel
movement analysis, live at [teupadel.com](https://www.teupadel.com).

## What's deployed

Three Helm charts, each a wrapper around
[`gitops.generic-app-chart`](https://github.com/cmoreira-dev/gitops.generic-app-chart)
(except `processor`, which is GPU-scheduled and has its own manifests):

| Chart | Deploys | Source repo |
|-------|---------|--------------|
| `helm/api` | `teupadel-api` — FastAPI orchestrator | [`api.ia.teupadel.com`](https://github.com/cmoreira-dev/api.ia.teupadel.com) |
| `helm/ui` | `teupadel-ui` — Next.js SSR frontend | [`ui.ia.teupadel.com`](https://github.com/cmoreira-dev/ui.ia.teupadel.com) |
| `helm/processor` | `teupadel-processor` — pose-estimation inference, GPU-scheduled | [`api.ia.pose-estimation`](https://github.com/cmoreira-dev/api.ia.pose-estimation) |

Reconciled by ArgoCD (`argocd/` folder), with `argocd-image-updater`
(in `gitops.core-addons`) writing back new image tags to `helm/*/values.yaml`
as CI publishes them to ECR.

## Networking

Ingress is NGINX Gateway Fabric (Gateway API) behind a Cloudflare tunnel,
provided by [`gitops.core-addons`](https://github.com/cmoreira-dev/gitops.core-addons).
The UI reaches the API over the in-cluster Service, not the public
domain.
