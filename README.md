# TP7 : Prometheus et Grafana sur Kubernetes avec Helm

Cluster utilisé : kind (Kubernetes 1.37) qui tourne dans Podman sur macOS.

## Installer la stack

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack \
  --version 45.7.1 \
  --namespace monitoring \
  --create-namespace
```

Vérifier que tout démarre :

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```
