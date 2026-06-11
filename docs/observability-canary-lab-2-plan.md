# Lab 2 moi - Viet Flask API co /metrics va build image

## Muc tieu

Tao mot API Python Flask don gian co endpoint:

- `/`: tra JSON thanh cong hoac loi gia lap.
- `/healthz`: endpoint health check.
- `/metrics`: endpoint Prometheus scrape metric.

Sau do build Docker image:

```text
w9-api:1
```

Va nap image vao Minikube profile:

```text
w9
```

Lab nay chua deploy API len Kubernetes. Lab 3 moi tao manifest `Rollout`, `Service`, `ServiceMonitor` va ArgoCD Application de deploy API.

## File se tao

Lab 2 se tao 2 file:

```text
app/app.py
app/Dockerfile
```

## File app/app.py se lam gi

Code Flask se:

- Tao app Flask.
- Bat metric Prometheus bang `prometheus-flask-exporter`.
- Doc bien moi truong `ERROR_RATE` de gia lap loi.
- Doc bien moi truong `VERSION` de biet ban dang chay.
- Endpoint `/` co the tra `200` hoac `500`.
- Endpoint `/healthz` tra `ok`.
- Endpoint `/metrics` duoc tao tu dong de Prometheus scrape.

Y tuong quan trong:

```text
ERROR_RATE=0   -> app gan nhu tot
ERROR_RATE=1   -> app loi 100%
VERSION=v1/v2  -> phan biet ban dang rollout
```

## File app/Dockerfile se lam gi

Dockerfile se:

- Dung image nen `python:3.12-slim`.
- Cai thu vien `flask` va `prometheus-flask-exporter`.
- Copy `app.py` vao image.
- Chay Flask tren port `8080`.

## Lenh build image

Sau khi tao file:

```powershell
docker build -t w9-api:1 app/
```

## Lenh nap image vao Minikube

Vi lab dung Minikube local, khong can push image len Docker Hub.

Nap image vao Minikube:

```powershell
minikube image load w9-api:1 -p w9
```

Kiem tra image da co trong Minikube:

```powershell
minikube image ls -p w9 | Select-String w9-api
```

## Test image local bang Docker tuy chon

Co the test nhanh bang Docker:

```powershell
docker run --rm -p 8088:8080 w9-api:1
```

Mo:

```text
http://localhost:8088/
http://localhost:8088/healthz
http://localhost:8088/metrics
```

Dung container bang `Ctrl + C`.

## Ket qua mong muon cua Lab 2

Dat khi:

- Co file `app/app.py`.
- Co file `app/Dockerfile`.
- Docker build thanh cong image `w9-api:1`.
- Minikube profile `w9` co image `w9-api:1`.
- App co endpoint `/metrics` de Prometheus dung o Lab 3.

Sau Lab 2, ta sang Lab 3: tao `k8s-api/` gom `Rollout`, `Service`, `ServiceMonitor` va ArgoCD Application `api`.

