# Work log / Nhật ký công việc

Mục tiêu của file này là ghi nhanh **bạn đã làm gì, khi nào, vì sao**, và **có gì cần bàn giao** cho người khác (dev/devops).

## 2026-03-17

- **Việc đã làm**
  - Tách repo thành 2 phần:
    - `dev/`: source code + tài liệu cho dev (`dev/src`, `dev/protos`, `dev/docs`)
    - `devops/`: manifest/deploy/IaC cho DevOps (`devops/kubernetes-manifests`, `devops/kustomize`, `devops/helm-chart`, `devops/istio-manifests`, `devops/terraform`, `devops/release`, `devops/.deploystack`)
  - Cập nhật đường dẫn quan trọng để phù hợp layout mới:
    - `README.md`, `skaffold.yaml`
    - GitHub Actions workflows trong `.github/workflows/*.yaml`
    - `dev/docs/development-guide.md`
    - `.gitignore`, `.github/workflows/README.md`

- **Ghi chú / lý do**
  - Tách layout giúp phân quyền và ownership rõ hơn: dev tập trung vào `dev/`, devops tập trung vào `devops/`.

- **Việc cần làm tiếp (TODO)**
  - Rà soát các tài liệu còn lại trong `dev/docs/` và `devops/` để sửa các link/đường dẫn cũ (nếu cần).
  - Chạy thử các lệnh chính:
    - `skaffold run`
    - `kubectl apply -f devops/release/kubernetes-manifests.yaml`

- **Lệnh đã chạy (tuỳ chọn)**
  - `git mv ...` (di chuyển thư mục để giữ lịch sử)

