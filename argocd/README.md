# Argo CD (GitOps)

This repo is the **manifests repository** in a 2-repo GitOps flow:

- **Code repo**: build/push images, then update manifests repo (image tag/digest)
- **Manifests repo (this)**: Argo CD watches and syncs to Kubernetes

## Environments

- **dev**: `onlineboutique-dev` (fast iteration, auto-sync ok)
- **staging**: `onlineboutique-staging` (pre-prod validation, auto-sync ok)
- **prod**: `onlineboutique-prod` (change controlled, usually manual sync)

## What to apply to the cluster

Apply these once to bootstrap Argo CD apps:

1) Update the `repoURL` placeholders in:
- `argocd/projects/onlineboutique-project.yaml`
- `argocd/apps/onlineboutique-rbac.yaml`
- `argocd/apps/onlineboutique-dev.yaml`
- `argocd/apps/onlineboutique-staging.yaml`
- `argocd/apps/onlineboutique-prod.yaml`

2) Apply:

```sh
kubectl apply -f argocd/projects/onlineboutique-project.yaml
kubectl apply -f argocd/apps/onlineboutique-rbac.yaml
kubectl apply -f argocd/apps/onlineboutique-dev.yaml
kubectl apply -f argocd/apps/onlineboutique-staging.yaml
kubectl apply -f argocd/apps/onlineboutique-prod.yaml
```

## Branching model (recommended)

Use **branch-per-environment** in the manifests repo:

- `main`: shared base + reviewed changes
- `env/dev`: what is deployed to dev
- `env/staging`: what is deployed to staging
- `env/prod`: what is deployed to prod

Promotion is done by PR merge:
`env/dev` -> `env/staging` -> `env/prod`.

