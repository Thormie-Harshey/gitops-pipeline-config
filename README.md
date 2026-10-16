# gitops-pipeline-config

The GitOps config repo for [gitops-pipeline-app](https://github.com/Thormie-Harshey/gitops-pipeline-app). ArgoCD watches the overlays here, so this repo is the single source of truth for what runs in each environment.

**Full documentation, workflow diagram and evidence:** see the [app repo README](https://github.com/Thormie-Harshey/gitops-pipeline-app#readme).

## Layout

```
base/                    Shared Deployment and Service
overlays/staging/        Namespace, image tag (updated by CI), 1 replica
overlays/production/     Namespace, image tag (updated by promotion PR), 3 replicas
argocd/                  AppProject, Applications, notifications (applied once by hand)
```

## Who changes what

| File | Changed by | Deployed by |
|---|---|---|
| `overlays/staging/kustomization.yaml` | CI bot, on every push to the app's `main` | ArgoCD, automatically |
| `overlays/production/kustomization.yaml` | A person, through a pull request | ArgoCD, after a manual Sync |

## Promotion (staging to production)

1. Verify the new version in staging.
2. Branch, copy the staging `newTag:` into `overlays/production/kustomization.yaml`, open a pull request.
3. Review, then **Squash and merge**. Production shows OutOfSync.
4. Click **Sync, Synchronize** on `gitops-pipeline-production` in ArgoCD.

## Rollback

```bash
git revert --no-edit <promotion commit SHA>
git push
```

Then sync production. History keeps both the release and the rollback.

Always run `git pull` before editing, because CI also commits here.
