# healthbot-infra

Infrastructure GitOps repository for k3s cluster operations.

## Repository scope
- Argo CD installation and platform settings
- Argo CD Image Updater settings
- Application manifests and environment overlays
- Ingress and routing for cluster workloads

Application source code and Helm chart live in a separate app repo:
- https://github.com/AndreyRubtsov/bot

## Layout
```
clusters/
  home-k3s/
    platform/
      argocd/
      argocd-image-updater/
    apps/
      healthbot/
      airflow/
```

## Bootstrap / upgrade Argo CD
```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update argo

kubectl create namespace argocd --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f clusters/home-k3s/platform/argocd/local-only-middleware.yaml

helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  -f clusters/home-k3s/platform/argocd/values.yaml
```

## Install Image Updater
```bash
kubectl -n argocd create secret generic argocd-image-updater-ghcr \
  --from-literal=creds='<GHCR_USER>:<GHCR_TOKEN>'

helm upgrade --install argocd-image-updater argo/argocd-image-updater \
  -n argocd \
  --version 0.14.0 \
  -f clusters/home-k3s/platform/argocd-image-updater/values.yaml
```

## Deploy apps
```bash
kubectl apply -f clusters/home-k3s/apps/airflow/ingress.yaml
kubectl apply -f clusters/home-k3s/apps/healthbot/application.yaml
```

## Change LAN URLs
Update values in:
- `clusters/home-k3s/apps/healthbot/values-lan.yaml`
- `clusters/home-k3s/platform/argocd/values.yaml`
- `clusters/home-k3s/apps/airflow/ingress.yaml`
