# Lab 3 moi - Viet k8s-api va de Prometheus thay metric

## Muc tieu

Deploy image `w9-api:1` da build o Lab 2 len Kubernetes bang Argo Rollouts, sau do cau hinh Prometheus scrape endpoint `/metrics`.

Ket qua mong muon:

- ArgoCD co them app `api`.
- Namespace `demo` co `Rollout api`, `Service api`, pod API.
- Prometheus target thay `api` la `UP`.
- Query Prometheus thay metric request cua Flask.

## File se tao

Lab 3 se tao 3 file:

```text
k8s-api/api.yaml
k8s-api/servicemonitor.yaml
argocd/apps/api.yaml
```

## File k8s-api/api.yaml se lam gi

File nay gom 2 resource:

- `Rollout api`: thay cho Deployment, do Argo Rollouts quan ly.
- `Service api`: tao dia chi on dinh cho pod API.

`Rollout api` se dung image:

```text
w9-api:1
```

Va cau hinh:

```yaml
ERROR_RATE: "0"
VERSION: "v1"
```

Y nghia:

- `ERROR_RATE=0`: app dang tot, khong inject loi.
- `VERSION=v1`: ban dau la phien ban v1.

Rollout co strategy canary:

```text
25% -> pause
50% -> pause 30s
100%
```

Lab 4 se dung strategy nay de promote/abort bang tay khi doi version.

## File k8s-api/servicemonitor.yaml se lam gi

File nay tao `ServiceMonitor` de Prometheus biet can scrape service `api`.

Prometheus se goi:

```text
http://api.demo.svc:8080/metrics
```

Moi 15 giay.

Neu khong co `ServiceMonitor`, app van chay nhung Prometheus se khong tu biet lay metric.

## File argocd/apps/api.yaml se lam gi

File nay tao ArgoCD Application ten `api`.

No tro vao:

```text
repoURL: https://github.com/mthanh12/gitops.git
path: k8s-api
targetRevision: main
destination namespace: demo
```

Viec tao file nay trong `argocd/apps/` giup app `root` tu quan ly app `api`.

## Luong hoat dong GitOps

Sau khi push/merge vao `main`:

```text
root
→ doc argocd/apps/api.yaml
→ tao Application api
→ api doc k8s-api/
→ tao Rollout + Service + ServiceMonitor
→ Prometheus scrape /metrics
```

## Lenh commit va push

```powershell
git add app/ k8s-api/ argocd/apps/api.yaml docs/observability-canary-lab-3-plan.md
git commit -m "api"
git push
```

Neu dang dung branch protection, push branch va tao PR vao `main`.

## Lenh kiem tra sau khi ArgoCD sync

Kiem tra Application:

```powershell
kubectl -n argocd get applications
```

Kiem tra Rollout, Service, Pod:

```powershell
kubectl -n demo get rollout,svc,pod
```

Kiem tra ServiceMonitor:

```powershell
kubectl -n demo get servicemonitor api
```

Tao traffic de metric tang:

```powershell
kubectl -n demo run load --image=busybox --restart=Never -- sh -c "while true; do wget -qO- api:8080/; done"
```

Mo Prometheus:

```powershell
kubectl -n monitoring port-forward svc/kube-prometheus-stack-prometheus 9090:9090
```

Query:

```text
flask_http_request_total{namespace="demo"}
```

Hoac xem target:

```text
Status -> Targets
```

Ky vong co target `api` trang thai `UP`.

## Ket qua mong muon cua Lab 3

Dat khi:

- App `api` tren ArgoCD la `Synced/Healthy`.
- Rollout `api` trong namespace `demo` la healthy.
- Prometheus target thay `api` la `UP`.
- Metric Flask tang khi tao traffic.

