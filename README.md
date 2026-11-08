# sample-app-deploy

Deployment config for [sample-app](https://github.com/andrewsgitacc/sample-app), promoted
**dev → staging → prod** by [Kargo](https://kargo.apps.lab.local) — Phase 10 of the
[platform lab](https://github.com/andrewsgitacc/labs/tree/main/platform).

```
 sample-app CI ──push :<sha>──► Harbor ──► Kargo Warehouse (new Freight)
                                               │
          ┌────────────────────────────────────┘
          ▼
   dev (automatic) ──► staging (Promote button) ──► prod (Promote button)
          │                    │                          │
   branch stage/dev     branch stage/staging       branch stage/prod
          │                    │                          │
   Argo CD sample-app-dev  ... -staging              ... -prod
```

## Layout

| Path | What |
|---|---|
| `base/` | Deployment, Service, HTTPRoute shared by every stage |
| `stages/<stage>/` | Kustomize overlay: namespace `sample-app-<stage>`, replicas, hostname |
| branch `stage/<stage>` | **Rendered** manifests for that stage, written only by Kargo |

You edit `main` (base and overlays) through pull requests as usual. A promotion takes the
Freight's image tag, renders `stages/<stage>` from `main` with it, commits the result to
`stage/<stage>` and syncs the Argo CD Application `sample-app-<stage>`, which deploys
that branch. So:

- `git log stage/prod` is the history of what ran in prod, and `git diff stage/staging
  stage/prod` shows exactly what promoting to prod will change.
- A change to `main` (say more replicas) reaches a stage only at its next promotion.

The Kargo project, its stages and the Argo CD Applications are defined in
[platform-gitops `kargo/sample-app/`](https://github.com/andrewsgitacc/platform-gitops/tree/main/kargo/sample-app).

## URLs

https://sample-app-dev.apps.lab.local · https://sample-app-staging.apps.lab.local ·
https://sample-app-prod.apps.lab.local (on the Mac: `curl -k`)

## CI

`.github/workflows/validate.yml` renders every stage and schema-checks it (kubeconform)
on pull requests.
