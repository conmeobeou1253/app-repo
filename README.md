# Demo App cho ArgoCD GitOps

Repository này chứa Kubernetes manifests cho một ứng dụng Nginx đơn giản, được quản lý bởi ArgoCD theo mô hình GitOps.

## 📁 Cấu trúc

```
app-repo/
├── deployment.yaml   # Nginx Deployment (1 replica)
├── service.yaml      # ClusterIP Service
└── README.md         # File này
```

## 🚀 Deploy bằng ArgoCD

### Cách 1: Dùng UI (Recommended cho người mới)

1. Mở ArgoCD UI tại `https://localhost:8080`
2. Đăng nhập với user `admin` và password từ command:
   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
   ```

3. Click **"+ New App"** và điền thông tin:
   - **Application Name**: `nginx-demo`
   - **Project**: `default`
   - **Sync Policy**: `Manual` (hoặc `Automatic` nếu muốn tự động sync)
   - **Repository URL**: `https://github.com/<your-username>/app-repo` (hoặc đường dẫn repo của bạn)
   - **Path**: `.` (root của repo)
   - **Cluster**: `https://kubernetes.default.svc`
   - **Namespace**: `default`

4. Click **"Create"** và sau đó click **"Sync"**

### Cách 2: Dùng kubectl (Pro tip)

Tạo file `argocd-app.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://github.com/<your-username>/app-repo'
    targetRevision: HEAD
    path: .
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

Apply:
```bash
kubectl apply -f argocd-app.yaml
```

## 🔄 Test GitOps Flow

1. **Thử tăng replicas**: Sửa `replicas: 1` thành `replicas: 3` trong `deployment.yaml`
2. Commit và push lên GitHub
3. Vào ArgoCD UI xem status → Click **"Refresh"** hoặc chờ auto-sync (3 phút)
4. Click **"Sync"** để apply thay đổi
5. Verify:
   ```bash
   kubectl get pods -l app=nginx-demo
   ```

## 🧪 Kiểm tra App

```bash
# Xem pods
kubectl get pods -l app=nginx-demo

# Xem service
kubectl get svc nginx-demo

# Test bằng port-forward
kubectl port-forward svc/nginx-demo 8081:80

# Mở browser: http://localhost:8081
```

## 📊 GitOps Benefits Demo

- ✅ **Single Source of Truth**: Mọi thay đổi đều từ Git
- ✅ **Audit Trail**: Git history = deployment history
- ✅ **Rollback dễ dàng**: `git revert` → ArgoCD auto-sync
- ✅ **Declarative**: Không cần `kubectl apply` manual

## 💡 Pro Tips

### Auto-sync
Để ArgoCD tự động sync mỗi 3 phút:
- Trong ArgoCD UI → App Details → **"Enable Auto-Sync"**

### Webhook (Instant sync)
Set up GitHub webhook để sync ngay khi có commit mới:
- Settings → Webhooks → Add webhook
- Payload URL: `https://<argocd-server>/api/webhook`

### Everything as Code
Thêm file `argocd-app.yaml` vào infra repo và apply qua Terraform:

```hcl
resource "kubernetes_manifest" "nginx_demo_app" {
  manifest = yamldecode(file("${path.module}/argocd-app.yaml"))
  
  depends_on = [helm_release.argocd]
}
```
