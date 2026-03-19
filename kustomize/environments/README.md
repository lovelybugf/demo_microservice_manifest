# Environments (Kustomize)

This directory provides **environment overlays** for Online Boutique.

Each overlay:
- deploys into a dedicated namespace (`onlineboutique-dev`, `onlineboutique-staging`, `onlineboutique-prod`)
- applies environment labels
- sets a reasonable default replica size per environment

## Apply

Prereqs (one-time):

```sh
kubectl apply -f rbac/namespaces.yaml
kubectl apply -f rbac/env/dev.yaml
kubectl apply -f rbac/env/staging.yaml
kubectl apply -f rbac/env/prod.yaml
```

Deploy the app:

```sh
# dev
kubectl apply -k kustomize/environments/dev

# staging
kubectl apply -k kustomize/environments/staging

# prod
kubectl apply -k kustomize/environments/prod
```

