# demo_microservice_devops

Kho chứa **manifests DevOps** cho demo microservice Online Boutique:
- Kubernetes manifests
- Kustomize (base + components + environment overlays)
- Helm chart
- Terraform
- Cấu hình RBAC, Argo CD, release bundles

---

## Cấu trúc dự án (Repo Manifests)

- `kubernetes-manifests/`  
  - Các manifest K8s “thô” (Deployment, Service, v.v.) cho từng microservice.
- `kustomize/`  
  - `kustomize/base/`: base manifests cho toàn bộ service (không gắn env/namespace).  
  - `kustomize/components/`: các thành phần tuỳ chọn (network policies, Istio, logging, v.v.).  
  - `kustomize/environments/{dev,staging,prod}/`: overlay cho từng môi trường, set namespace + replicas + labels.
- `helm-chart/`  
  - Helm chart đóng gói toàn bộ ứng dụng (có thể dùng thay cho Kustomize nếu muốn).
- `rbac/`  
  - `namespaces.yaml`: tạo 3 namespace `onlineboutique-dev/staging/prod` + labels môi trường.  
  - `env/{dev,staging,prod}.yaml`: RoleBinding theo **group** (`onlineboutique-devs`, `onlineboutique-qa`, `onlineboutique-devops`) gắn với ClusterRole `edit/view/admin`.
- `argocd/`  
  - `projects/onlineboutique-project.yaml`: định nghĩa `AppProject` Online Boutique.  
  - `apps/onlineboutique-{rbac,dev,staging,prod}.yaml`: mỗi file là 1 `Application` Argo CD, trỏ tới branch + path tương ứng.
- `terraform/`  
  - Hạ tầng cloud (cluster, network, v.v.) được mô tả bằng Terraform.
- `release/`  
  - Bundle manifest / YAML đã render sẵn phục vụ phát hành.
- `huong-dan-anh/`  
  - **Thư mục trống để bạn chèn ảnh hướng dẫn** (screenshot Argo CD, Rancher, kubectl, v.v.).

Repo mã nguồn ứng dụng: `demo_microservice_code` (GitHub: `lovelybugf/demo_microservice_code`).

---

## Mô hình GitOps / Argo CD (2 repo)

Repo này được thiết kế làm **manifests repository** trong mô hình 2 repo:

- **Repo code** (`demo_microservice_code`):
  - Build và push Docker image.
- **Repo manifests** (`demo_microservice_devops` / GitHub: `lovelybugf/demo_microservice_manifest`):
  - Lưu toàn bộ manifests/Kustomize/Helm/RBAC/Argo CD.
  - Được Argo CD theo dõi và sync xuống cluster.

Sơ đồ tổng thể luồng GitOps (đề xuất):

> _Bạn có thể lưu một ảnh sơ đồ luồng vào `huong-dan-anh/gitops-flow.png` và hiển thị trong README như sau:_  
> `![Luồng GitOps 2 repo](huong-dan-anh/gitops-flow.png)`

Các phần chính:

- `rbac/`:
  - Tạo namespace:
    - `onlineboutique-dev`
    - `onlineboutique-staging`
    - `onlineboutique-prod`
  - Bind group từ IdP:
    - `onlineboutique-devs` → ClusterRole `edit`
    - `onlineboutique-qa` → ClusterRole `view`
    - `onlineboutique-devops` → ClusterRole `admin`
- `kustomize/base/`:
  - Định nghĩa phiên bản “trung tính” của tất cả service.
- `kustomize/environments/{dev,staging,prod}/`:
  - Trỏ tới base, set namespace đúng (`onlineboutique-*`), chỉnh replicas/labels theo env.
- `argocd/`:
  - `AppProject` + `Application` cho:
    - Bootstrap RBAC (`main` branch → `rbac/`).
    - Mỗi env:
      - `env/dev` → `kustomize/environments/dev`
      - `env/staging` → `kustomize/environments/staging`
      - `env/prod` → `kustomize/environments/prod`

Nhờ tách repo:

- Code và manifests sạch sẽ, dễ phân quyền.
- Promotion môi trường qua Git branch/PR.
- Argo CD là “single source of truth” cho trạng thái triển khai.

---

## Quy trình cập nhật hàng ngày (nhánh + Argo CD)

### Nhánh sử dụng

- `main`  
  - Nguồn chuẩn cho manifests dùng chung (base, components, Argo CD apps, RBAC, v.v.).
- `env/dev`  
  - Phản ánh trạng thái đang chạy ở môi trường **dev**.
- `env/staging`  
  - Phản ánh trạng thái đang chạy ở môi trường **staging**.
- `env/prod`  
  - Phản ánh trạng thái đang chạy ở môi trường **production**.

### Bước 1 – Làm việc trên `main`

1. Tạo/merge PR vào `main` để:
   - Sửa manifests
   - Thêm/bật components (network policies, Istio, v.v.)
   - Điều chỉnh Kustomize overlays
2. Sau khi review/OK → code mới nằm trên `main`.

### Bước 2 – Đẩy thay đổi sang `env/dev` (dev)

```bash
git checkout env/dev
git merge main
git push
```

- Argo CD application `onlineboutique-dev` (watching `env/dev`) sẽ detect commit mới và sync vào namespace `onlineboutique-dev`.

### Bước 3 – Promote `env/dev` → `env/staging`

```bash
git checkout env/staging
git merge env/dev
git push
```

- `onlineboutique-staging` (watching `env/staging`) sẽ sync vào namespace `onlineboutique-staging`.

### Bước 4 – Promote `env/staging` → `env/prod`

```bash
git checkout env/prod
git merge env/staging
git push
```

- `onlineboutique-prod` (watching `env/prod`) sẽ thấy commit mới.  
- Thông thường **prod để manual sync**:
  - Vào UI Argo CD → chọn app `onlineboutique-prod` → bấm **Sync** khi đã sẵn sàng release.

---

## Hướng dẫn triển khai từng phần

### 1. Bootstrap RBAC + namespace

1. Đảm bảo IdP/Rancher có các group:
   - `onlineboutique-devs`
   - `onlineboutique-qa`
   - `onlineboutique-devops`
2. Apply RBAC (có thể để Argo CD app `onlineboutique-rbac` handle, hoặc dùng `kubectl apply -f rbac/` lần đầu).

> _Chỗ này có thể chèn ảnh màn hình cấu hình group trong Rancher/IdP._  
> (Thêm file vào `huong-dan-anh/` rồi nhúng vào README.)

### 2. Bootstrap Argo CD Applications

1. Cài sẵn Argo CD trên cluster (qua Rancher/YAML/Helm).
2. Apply các file trong `argocd/`:
   ```bash
   kubectl apply -f argocd/projects/onlineboutique-project.yaml
   kubectl apply -f argocd/apps/onlineboutique-rbac.yaml
   kubectl apply -f argocd/apps/onlineboutique-dev.yaml
   kubectl apply -f argocd/apps/onlineboutique-staging.yaml
   kubectl apply -f argocd/apps/onlineboutique-prod.yaml
   ```
3. Vào UI Argo CD để kiểm tra 4 application.

> _Chỗ này có thể chèn ảnh màn hình UI Argo CD hiển thị 4 app._  

### 3. Áp dụng Kustomize theo môi trường

- Dev:
  - Argo CD app `onlineboutique-dev` → `env/dev` → `kustomize/environments/dev`.
- Staging:
  - `onlineboutique-staging` → `env/staging` → `kustomize/environments/staging`.
- Prod:
  - `onlineboutique-prod` → `env/prod` → `kustomize/environments/prod` (manual sync).

Có thể test local bằng:

```bash
kubectl kustomize kustomize/environments/dev
```

để xem YAML được render.

### 4. CI cập nhật manifests từ repo code

Trong pipeline CI của repo code:

1. Build & push image.
2. Clone repo manifests `demo_microservice_devops`.
3. Update tag/digest trong:
   - Kustomize component/tag
   - Hoặc trực tiếp Deployment tương ứng.
4. Commit vào `env/dev` và push:
   ```bash
   git checkout env/dev
   # chỉnh file
   git add .
   git commit -m "ci: bump image <service> to <tag>"
   git push
   ```

> _Chỗ này bạn có thể chèn ảnh pipeline CI chạy thành công._  

