# AIOps GitOps Manifests

Repo này là nguồn sự thật cho trạng thái deploy Kubernetes của AIOps video platform.

Repo source `aiops-multi-agent-rag-k8s` chịu trách nhiệm build/test/scan image. Repo này chịu trách nhiệm nói cluster cần chạy image tag nào, namespace nào, config nào và manifest nào. Argo CD sync repo này, không deploy trực tiếp từ CI bằng `kubectl apply`.

## Structure

```text
aiops-gitops-manifests/
├── README.md
├── argocd-apps/
│   ├── root-app.yaml
│   ├── app-dev.yaml
│   ├── app-demo.yaml
│   └── observability.yaml
│
├── environments/
│   ├── dev/
│   │   ├── namespace.yaml
│   │   ├── kustomization.yaml
│   │   ├── identity-service.yaml
│   │   ├── video-service.yaml
│   │   ├── media-worker.yaml
│   │   └── values/
│   └── demo/
│       ├── namespace.yaml
│       ├── kustomization.yaml
│       └── values/
│
└── platform/
    ├── ingress/
    ├── cert-manager/
    ├── monitoring/
    ├── logging/
    ├── storage/
    └── registry/
```

## GitOps Flow

```text
Developer merge code
→ GitLab CI validate + scan + build image
→ Push image lên registry
→ CI tạo commit/MR cập nhật image tag trong repo này
→ Merge GitOps MR
→ Argo CD sync Kubernetes
→ Observability ghi nhận rollout
→ AIOps Deployment Agent dùng GitOps diff làm RCA evidence
```

## Bootstrap

Sau khi cài Argo CD và cấu hình repo access:

```bash
kubectl apply -n argocd -f argocd-apps/root-app.yaml
```

Nếu repo private, add Git credentials vào Argo CD trước khi sync.

## Notes

- Manifest service ở `environments/dev` đang để `replicas: 0` cho tới khi image thật sẵn sàng.
- Không commit secret plaintext.
- Secret nên đi qua External Secrets, Sealed Secrets hoặc SOPS.
- Rollback bằng Git revert trong repo này.
