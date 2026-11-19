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

## Déployer l'API FastAPI du TP6

L'image vient du [TP6](https://github.com/arthaud-proust-junia-isen/tp6-monitoring-fastapi). Elle n'est publiée sur aucun registre, donc on la construit avec Podman et on la charge directement dans le nœud kind :

```bash
podman build -t localhost/fastapi-app:tp6 ../tp6
podman save localhost/fastapi-app:tp6 -o fastapi.tar
KIND_EXPERIMENTAL_PROVIDER=podman kind load image-archive fastapi.tar --name tp
```

Puis on applique les manifestes :

```bash
kubectl apply -f k8s/
```

Le ServiceMonitor `fastapi-monitor` porte le label `release: prometheus`, sinon l'opérateur ne le donne pas à Prometheus. Dans Status > Targets, les trois pods doivent être `UP`, et cette requête doit répondre :

```promql
http_requests_total{method="GET",path="/metrics",status="200"}
```

## Dashboard Grafana

`k8s/grafana-dashboard.yaml` est un ConfigMap avec le label `grafana_dashboard: "1"`. Le sidecar de Grafana le détecte et crée le dashboard « FastAPI » tout seul, sans passer par l'interface. Il affiche le nombre total de requêtes, les pods scrapés, le taux d'erreurs, les requêtes par seconde et la latence P95.
