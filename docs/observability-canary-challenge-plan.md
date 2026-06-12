# Challenge - SLO, email alert, auto-abort

## Muc tieu

Hoan thien pipeline an toan cho app `api`:

- GitOps: moi thay doi van di qua Git va ArgoCD.
- Observability: Prometheus co SLO dua tren ti le loi HTTP 5xx.
- Alert: khi inject loi lam error rate vuot nguong, Alertmanager gui email ca nhan.
- Canary tu dong: Argo Rollouts dung `AnalysisTemplate`; ban tot di tiep, ban loi tu abort.

## File se tao/sua

- `argocd/apps/kube-prometheus-stack.yaml`
  - Cho Prometheus doc `PrometheusRule` tu namespace khac.
  - Cho Alertmanager doc cau hinh SMTP tu Secret `alertmanager-email-config`.
- `k8s-api/slo-alert.yaml`
  - `AnalysisTemplate api-success-rate`: query Prometheus, yeu cau success rate >= 95%.
  - `PrometheusRule api-slo`: alert `ApiHighErrorRate` khi 5xx > 5% trong 1 phut.
- `k8s-api/api.yaml`
  - Thay `pause` tay bang buoc `analysis` de canary tu quyet dinh.
- `docs/alertmanager-email-secret.md`
  - Lenh tao Secret email ngoai Git de khong lo mat khau.

## Luu y ve email

Khong nen commit mat khau Gmail that vao Git. Truoc khi test email that,
tao Secret rieng trong cum theo file `docs/alertmanager-email-secret.md`.

Mau lenh rut gon:

```powershell
kubectl -n monitoring get secret alertmanager-email-config
```

## Cach test

1. Push branch va merge PR vao `main`.
2. Sync app `kube-prometheus-stack` va `api` trong ArgoCD.
3. Tao traffic:

```powershell
kubectl -n demo run load --image=busybox --restart=Never -- `
  sh -c "while true; do wget -qO- http://api:8080/; sleep 1; done"
```

4. Inject loi bang Git: sua `k8s-api/api.yaml`:

```yaml
- name: ERROR_RATE
  value: "0.5"
- name: VERSION
  value: "v3-bad"
```

5. Commit, push, merge. Canary se chay analysis.

Ket qua mong doi:

- Prometheus alert `ApiHighErrorRate` chuyen firing.
- Alertmanager gui email.
- Rollout loi bi abort, traffic quay ve stable ReplicaSet cu.
