# Lab 4 moi - Canary thu cong voi Argo Rollouts

## Muc tieu

Thuc hanh canary bang tay cho app `api`.

Ta se doi bien moi truong:

```text
VERSION: v1 -> v2
```

trong file:

```text
k8s-api/api.yaml
```

Sau khi thay doi di qua Git va ArgoCD sync, Argo Rollouts se bat dau canary. Rollout se khong dua version moi len 100% ngay, ma dung o buoc 25% de cho minh quan sat va quyet dinh.

## File se sua

Lab 4 chi sua 1 file:

```text
k8s-api/api.yaml
```

Noi dung thay doi:

```yaml
- name: VERSION
  value: "v1"
```

thanh:

```yaml
- name: VERSION
  value: "v2"
```

Khong build image moi trong Lab 4. Image `w9-api:1` da doc bien moi truong `VERSION`, nen doi YAML la du de pod moi tra ve version moi.

## Vi sao doi VERSION lai kich canary

`VERSION` nam trong pod template cua `Rollout`.

Khi pod template thay doi, Argo Rollouts se tao ReplicaSet moi cho version `v2`.

Voi strategy canary hien co:

```yaml
strategy:
  canary:
    steps:
      - setWeight: 25
      - pause: {}
      - setWeight: 50
      - pause:
          duration: 30s
      - setWeight: 100
```

Rollout se:

1. Tao pod version `v2`.
2. Dua mot phan traffic/pod sang v2.
3. Dung lai o buoc pause sau 25%.
4. Cho nguoi van hanh quan sat metric/log.
5. Neu tot thi promote.
6. Neu te thi abort ve version cu.

## Lenh commit va push

```powershell
git add k8s-api/api.yaml docs/observability-canary-lab-4-plan.md
git commit -m "api v2"
git push
```

Neu dang dung branch protection, tao PR vao `main`, cho CI pass roi merge.

## Lenh quan sat rollout

Sau khi ArgoCD sync:

```powershell
kubectl argo rollouts get rollout api -n demo --watch
```

Neu chua co plugin `kubectl argo rollouts`, co the dung:

```powershell
kubectl -n demo describe rollout api
kubectl -n demo get rs,pod -l app=api
```

## Promote neu ban moi tot

Khi thay canary tot:

```powershell
kubectl argo rollouts promote api -n demo
```

Neu rollout con pause tiep o 50%, promote lan nua:

```powershell
kubectl argo rollouts promote api -n demo
```

Ket qua mong muon:

```text
VERSION v2 len 100%
Rollout api Healthy
```

## Abort neu ban moi te

Neu thay loi, huy canary:

```powershell
kubectl argo rollouts abort api -n demo
```

Ket qua:

```text
Rollout quay ve ReplicaSet stable cu
Version v1 tiep tuc phuc vu
```

## Kiem tra version cua API

Mo API:

```powershell
kubectl -n demo port-forward svc/api 8089:8080
```

Goi:

```text
http://localhost:8089/
```

Trong canary co the thay luc tra `v1`, luc tra `v2` tuy pod nao nhan request.

## Ket qua mong muon cua Lab 4

Dat khi:

- Doi `VERSION` qua Git, khong sua tay cluster.
- ArgoCD sync thay doi vao Rollout.
- Argo Rollouts tao canary ReplicaSet moi.
- Rollout dung o pause 25%.
- Co the promote len tiep hoac abort bang lenh tay.

