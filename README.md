# Demo App cho ArgoCD GitOps (Helm-based)

Repository này chứa Helm chart để demo GitOps chuẩn Enterprise với ArgoCD.

## 📁 Cấu trúc

```
app-repo/
├── chart/                    # Helm chart
│   ├── Chart.yaml
│   ├── values.yaml           # Dev/Staging (mặc định)
│   ├── values-prod.yaml      # Prod override (replica 3, res cao hơn)
│   └── templates/
│       ├── deployment.yaml
│       └── service.yaml
├── argocd-app-helm.yaml      # ArgoCD Application (dùng Helm + values-prod)
└── README.md
```

## 🚀 Deploy bằng ArgoCD (Helm)

### Cách 1: UI
1) Port-forward ArgoCD UI: `kubectl port-forward svc/argocd-server -n argocd 8080:443`
2) Mở `https://localhost:8080`, login `admin` + password từ:
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
3) + New App:
   - Name: `nginx-demo-helm`
   - Repo URL: `https://github.com/<your-username>/app-repo`
   - Path: `chart`
   - Cluster: `https://kubernetes.default.svc`
   - Namespace: `default`
   - Value files: `values-prod.yaml`
4) Create → Sync.

### Cách 2: GitOps thuần (khuyên dùng)
1) Sửa repoURL trong `argocd-app-helm.yaml` cho đúng GitHub của bạn.
2) Apply:
```bash
kubectl apply -f argocd-app-helm.yaml
```

ArgoCD sẽ render Helm chart với `values-prod.yaml` và deploy vào namespace `default`.

## 🔄 Test GitOps Flow (Helm)

1) Mở `chart/values-prod.yaml`, chỉnh:
   - `replicaCount: 3 -> 5` (ví dụ)
   - Hoặc đổi image tag: `image.tag: 1.27-alpine -> 1.27.3-alpine`
2) Commit + push lên Git.
3) ArgoCD sẽ phát hiện thay đổi → Sync (auto hoặc bấm Sync).
4) Kiểm tra:
```bash
kubectl get pods -l app=nginx-demo-helm
```

## 🧪 Kiểm tra App

```bash
kubectl get svc nginx-demo-helm
kubectl port-forward svc/nginx-demo-helm 8081:80
# Browser: http://localhost:8081
```

## 💡 Lưu ý cho Portfolio

- Helm tách logic (templates) và config (values). Dev/Stg/Prod chỉ khác values.
- CI/CD update image tag bằng `yq` vào `values-prod.yaml`, commit, ArgoCD tự sync.
- argocd-app-helm.yaml là Everything-as-Code, phù hợp show GitOps mindset.
