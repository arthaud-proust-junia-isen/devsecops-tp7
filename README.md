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

## Accéder à Prometheus et Grafana

```bash
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Prometheus répond sur http://localhost:9090 et Grafana sur http://localhost:3000 (`admin` / `prom-operator`). Si le mot de passe a changé, il est dans le secret de Grafana :

```bash
kubectl get secret -n monitoring prometheus-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

Pour tester que les métriques du cluster remontent, la requête `kube_pod_container_status_running` doit renvoyer `1` pour chaque conteneur.
