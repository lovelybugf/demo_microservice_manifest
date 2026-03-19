# demo_microservice_devops

DevOps repo for Online Boutique (Kubernetes manifests, Kustomize components, Helm chart, Terraform, release artifacts).

## Layout
- `kubernetes-manifests/`: raw manifests (Deployment/Service…) for the microservices.
- `kustomize/`: base manifests + optional components + environment overlays (`dev`, `staging`, `prod`).
- `helm-chart/`: Helm chart packaging the application (alternative to Kustomize).
- `rbac/`: namespaces and RBAC configuration (group-based access for dev/qa/devops across envs).
- `argocd/`: Argo CD `AppProject` and `Application` definitions for GitOps deployment per environment.
- `terraform/`: IaC for provisioning cloud infrastructure.
- `release/`: release bundles / pre-rendered manifests.

## Related
- Application source code lives in a separate repo: `demo_microservice_code`.

## GitOps / Argo CD model

This repo is designed to be the **manifests repository** in a two-repo GitOps workflow:

- The **code repo** builds and pushes Docker images.
- CI then updates this **manifests repo** (for example, image tags/digests in Kustomize overlays).
- **Argo CD** watches this repo and syncs the desired state to the Kubernetes cluster.

Key pieces for this flow:

- `rbac/` creates three namespaces:
  - `onlineboutique-dev`
  - `onlineboutique-staging`
  - `onlineboutique-prod`
  and binds identity provider groups:
  - `onlineboutique-devs` → `edit`
  - `onlineboutique-qa` → `view`
  - `onlineboutique-devops` → `admin`
  in each namespace.
- `kustomize/base/` defines a neutral version of all services (no env/namespace baked in).
- `kustomize/environments/{dev,staging,prod}/`:
  - point to the base
  - set the target namespace
  - adjust replicas and labels per environment.
- `argocd/` defines:
  - one `AppProject` for Online Boutique
  - one `Application` for RBAC/bootstrap
  - one `Application` per environment:
    - `env/dev` branch → `kustomize/environments/dev`
    - `env/staging` branch → `kustomize/environments/staging`
    - `env/prod` branch → `kustomize/environments/prod`

This separation lets you:

- keep application code and infrastructure manifests in different repositories
- promote changes between environments via Git branches/PRs
- enforce least-privilege RBAC for different teams (dev, QA, platform)
- use Argo CD as the single source of truth for what is actually deployed.
