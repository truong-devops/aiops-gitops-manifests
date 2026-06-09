# AIOps GitOps Manifests

Repo này là nguồn sự thật cho trạng thái triển khai Kubernetes của dự án AIOps video platform.

Repo source code build image và chạy test. Repo GitOps này quyết định cluster đang chạy image tag nào, replica nào, config nào và manifest nào. Argo CD sẽ watch repo này rồi sync Kubernetes theo Git.

## Vai Trò

- Lưu Kubernetes manifests theo môi trường.
- Tách cấu hình deploy khỏi source code.
- Cho phép rollback bằng Git revert.
- Tạo audit trail cho thay đổi image tag/config.
- Cung cấp evidence cho Deployment Agent khi làm RCA.

## Cấu Trúc

```text
aiops-gitops-manifests/
├── bootstrap/
│   └── root-app.yaml
├── argocd-apps/
│   ├── apps-dev.yaml
│   ├── apps-demo.yaml
│   ├── platform-dev.yaml
│   └── platform-demo.yaml
├── apps/
│   ├── base/
│   └── overlays/
│       ├── dev/
│       └── demo/
├── platform/
│   ├── base/
│   └── overlays/
│       ├── dev/
│       └── demo/
├── environments/
│   ├── dev/
│   └── demo/
├── clusters/
│   └── local/
└── scripts/
```

## Nguyên Tắc

- Argo CD sync từ repo này, không deploy trực tiếp bằng `kubectl apply` từ CI.
- CI ở repo source code chỉ build image, push registry, rồi cập nhật image tag trong repo GitOps.
- Mỗi môi trường có overlay riêng: `dev`, `demo`.
- `apps/` chứa manifest cho ứng dụng.
- `platform/` chứa hạ tầng dùng chung như ingress, observability, storage, registry.
- `bootstrap/root-app.yaml` chỉ dùng để cài app-of-apps ban đầu.

## Luồng GitOps

```text
Developer merge code
→ GitLab CI test/scan/build image
→ Push image lên Harbor/GitHub Container Registry
→ CI tạo commit/MR cập nhật image tag trong repo này
→ Merge GitOps MR
→ Argo CD sync Kubernetes
→ Observability ghi nhận rollout mới
→ AIOps Deployment Agent dùng GitOps diff làm evidence RCA
```

## Kết Nối GitHub

Trên GitHub, tạo repo:

- Owner: `truong-devops`
- Repository name: `aiops-gitops-manifests`
- Description: `GitOps manifests for AIOps Kubernetes video platform`
- Visibility: public hoặc private đều được.
- Add README: off.
- Add .gitignore: no.
- Add license: no.

Sau khi bấm **Create repository**, chạy:

```bash
cd ~/workspace/aiops-platform-workspace/aiops-gitops-manifests

git branch -M main
git remote add origin git@github.com:truong-devops/aiops-gitops-manifests.git

git add .
git commit -m "chore: scaffold gitops manifests"
git push -u origin main
```

Nếu muốn dùng HTTPS giống repo source code hiện tại:

```bash
cd ~/workspace/aiops-platform-workspace/aiops-gitops-manifests

git branch -M main
git remote add origin https://github.com/truong-devops/aiops-gitops-manifests.git

git add .
git commit -m "chore: scaffold gitops manifests"
git push -u origin main
```

Nếu chưa cấu hình SSH GitHub:

```bash
ssh -T git@github.com
```

Nếu báo chưa có key, tạo key mới:

```bash
ssh-keygen -t ed25519 -C "your-email@example.com"
pbcopy < ~/.ssh/id_ed25519.pub
```

Sau đó vào GitHub:

```text
Settings → SSH and GPG keys → New SSH key
```

Dán public key rồi thử lại:

```bash
ssh -T git@github.com
git push -u origin main
```

## Bootstrap Argo CD

Sau khi cài Argo CD vào cluster và cấu hình repo access, apply root app:

```bash
kubectl apply -n argocd -f bootstrap/root-app.yaml
```

Root app sẽ quản lý các child app trong `argocd-apps/`.

Nếu repo private, cần add repo credential vào Argo CD trước khi sync.

## Ghi Chú Mở Rộng

- Khi có service mới, thêm manifest vào `apps/base/<service-name>/`.
- Khi cần khác nhau giữa `dev` và `demo`, thêm patch trong `apps/overlays/<env>/`.
- Khi cần thêm platform component, thêm vào `platform/base/` rồi bật/tắt trong overlay.
- Không commit secret plaintext. Dùng External Secrets, Sealed Secrets hoặc SOPS nếu cần quản lý secret bằng GitOps.
