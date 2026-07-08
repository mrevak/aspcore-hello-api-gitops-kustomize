# aspcore-hello-api-gitops-kustomize (GitOps repo — Kustomize engine)

GitOps manifests for the **Kustomize** deploy variant. A shared `base/` plus a
thin overlay per environment; cocicd bumps the `newTag` in each overlay's
`kustomization.yaml` `images:` block.

## Layout

```
base/
  deployment.yaml       # 2 replicas, container aspcore-hello-api
  service.yaml
  kustomization.yaml    # resources: deployment.yaml, service.yaml
environments/
  staging/kustomization.yaml   # resources: ../../base ; images[].newTag (cocicd edits) — ns: kust-staging
  rc/kustomization.yaml        # ns: kust-rc
argocd/
  staging-application.yaml     # Application -> environments/staging (kustomize auto-detected)
  rc-application.yaml
```

cocicd deploy config (in the projsettings-kustomize repo):
`engine: kustomize`, `updates: [{ name: localhost:5000/library/aspcore-hello-api, new_tag: ... }]`.
The updater matches `images[].name` and sets `newTag`; it also injects
`cocicd.io/*` keys under `commonAnnotations`.

## Setup

```bash
git init && git add . && git commit -m "kustomize gitops"
git remote add origin git@github.com:mrevak/aspcore-hello-api-gitops-kustomize.git
git push -u origin main

kubectl apply -f argocd/staging-application.yaml
kubectl apply -f argocd/rc-application.yaml
kubectl get pods -n kust-staging

# sanity-check the render locally:
kubectl kustomize environments/staging
```

> Uses its own namespaces (`kust-staging` / `kust-rc`) and ArgoCD app names so it
> can coexist with the yaml/helm variants. Private repo → add ArgoCD repo creds or
> make it public.