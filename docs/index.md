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

## API runtime: migrations and secrets

- **Migrations:** the API Deployment runs `python -m migrate` in an
  `initContainer` (same image, same `securityContext`) before any replica serves
  traffic; an advisory lock serializes concurrent replicas. If it fails the pod
  stays in `Init:Error` and the previous ReplicaSet keeps serving.
- **Secrets:** the `teupadel-api-secret` ExternalSecret reads SSM parameters
  `/teupadel/google/oauth/{client_id,client_secret}`, `/teupadel/api/session-secret-key`
  (must be identical across replicas), `/teupadel/ses/api` (SES credentials) and
  the LiteLLM key under `/homelab/`. The `ClusterSecretStore` user
  (`external-secrets-operator`) needs its IAM policy to allow both
  `parameter/homelab/*` and `parameter/teupadel/*` — a missing path shows as
  `SecretSyncedError` and the pod silently loses those variables.
- **Database:** credentials come from the CNPG-generated `teupadel-app` Secret
  (`PGHOST`/`PGUSER`/…), database `teupadeldb`.

## Networking

Ingress is NGINX Gateway Fabric (Gateway API) behind a Cloudflare tunnel,
provided by [`gitops.core-addons`](https://github.com/cmoreira-dev/gitops.core-addons).
The UI reaches the API over the in-cluster Service, not the public
domain.
