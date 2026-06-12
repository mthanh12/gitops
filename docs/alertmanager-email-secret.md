# Alertmanager email secret

Khong commit Gmail App Password vao Git vi repo public. Tao Secret nay truc tiep trong cum truoc khi sync `kube-prometheus-stack`.

```powershell
@'
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-email-config
  namespace: monitoring
  annotations:
    argocd.argoproj.io/sync-wave: "0"
type: Opaque
stringData:
  alertmanager.yaml: |
    global:
      resolve_timeout: 5m
      smtp_smarthost: "smtp.gmail.com:587"
      smtp_from: "nguyeminhthanh1210@gmail.com"
      smtp_auth_username: "nvtvlog234@gmail.com"
      smtp_auth_password: "GMAIL_APP_PASSWORD_CUA_BAN"
      smtp_require_tls: true

    route:
      receiver: personal-email
      group_by: ["alertname", "service", "severity"]
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 4h

    receivers:
      - name: personal-email
        email_configs:
          - to: "nguyeminhthanh1210@gmail.com"
            send_resolved: true

    inhibit_rules: []
'@ | kubectl apply -f -
```

Sau khi tao Secret, sync app `kube-prometheus-stack` de Alertmanager doc cau hinh email.
