# RBAC for Online Boutique environments

This directory contains Kubernetes RBAC manifests using **group-based** access control for three namespaces:

- `onlineboutique-dev`
- `onlineboutique-staging`
- `onlineboutique-prod`

## Group model (recommended)

Configure your cluster's identity provider so users authenticate as members of these groups:

- **Developers**: `onlineboutique-devs`
- **QA**: `onlineboutique-qa`
- **DevOps / Platform**: `onlineboutique-devops`

The manifests bind these groups to Kubernetes built-in ClusterRoles (`view`, `edit`, `admin`) via **namespaced RoleBindings**.

## Apply (kubectl)

Create namespaces (one-time):

```sh
kubectl apply -f rbac/namespaces.yaml
```

Then apply RBAC per environment:

```sh
# dev
kubectl apply -f rbac/env/dev.yaml

# staging
kubectl apply -f rbac/env/staging.yaml

# prod
kubectl apply -f rbac/env/prod.yaml
```

## GitOps (Option A branch-per-env)

- Put shared RBAC changes through PRs into `main`.
- Promote to environments by merging into:
  - `env/dev`
  - `env/staging`
  - `env/prod`

