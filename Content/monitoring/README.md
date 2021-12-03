# Stack de Monitoramento

- [`Prometheus`](https://prometheus.io/)
- [`AlertManager`](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [`Node Exporter`](https://github.com/prometheus/node_exporter)
- [`Grafana`](https://grafana.com/oss/grafana/)
- [`Loki`](https://grafana.com/oss/loki/)
- [`Promtail`](https://grafana.com/docs/loki/latest/clients/promtail/)

## Prometheus / AlertManager / Node Exporter / Push Gateway

Adicione o repositório contendo os `Helm Charts`.
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Implante a stack com o seguinte comando:
```bash
helm upgrade --install prometheus prometheus-community/prometheus --version 15.0.1 --namespace monitoring --values prometheus/prometheus.yaml
```
> Confira o arquivo [`prometheus.yaml`](./prometheus/prometheus.yaml) e ajuste conforme a sua necessidade.

Para ajustar algo na stack após provisionado, basta editar o arquivo [`prometheus.yaml`](./prometheus/prometheus.yaml) e executar o comando de `upgrade` abaixo:
```bash
helm upgrade prometheus prometheus-community/prometheus --version 15.0.1 --namespace monitoring --values prometheus/prometheus.yaml
```

## Grafana

Adicione o repositório contendo os Helm Charts.
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

## Grafana Loki

## Promtail