# Kamerton examples

Ready-made setups for trying [Kamerton](https://github.com/kamerton-labs/kamerton),
a lightweight observability orchestrator on top of Prometheus, Alertmanager,
Grafana, Loki, Tempo and the OpenTelemetry Collector.

| Folder | What it is |
| --- | --- |
| [otel-demo](otel-demo) | Kamerton next to the [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo) (an open-source shop of about fifteen services with switchable faults): install values and a system description |
| [second-prometheus](second-prometheus) | A second Prometheus on the same cluster, to try several sets of programs |

Nothing here contains secrets. Chat IDs and email addresses in the
descriptions are placeholders.

## Shop demo on a local cluster

You need a small Kubernetes cluster (kind, k3s, colima), `kubectl`, `helm`
and `docker`. Clone both repositories side by side:

```sh
git clone https://github.com/kamerton-labs/kamerton.git
git clone https://github.com/kamerton-labs/examples.git
cd kamerton
```

1. Build the image and install Kamerton with the whole stack:

   ```sh
   docker build -t kamerton:dev .
   # make the image visible to the cluster, for example:
   #   kind load docker-image kamerton:dev
   helm install kamerton ./deploy/helm/kamerton -n kamerton --create-namespace \
     --dependency-update -f ../examples/otel-demo/values-kamerton.yaml
   ```

2. Install the shop. Its own backends are off; all telemetry goes to
   Kamerton's Collector:

   ```sh
   helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
   helm upgrade --install otel-demo open-telemetry/opentelemetry-demo \
     --version 0.42.6 -n otel-demo --create-namespace \
     -f ../examples/otel-demo/values-demo.yaml
   ```

3. Describe the shop to Kamerton and open the UI:

   ```sh
   kubectl -n kamerton apply -f ../examples/otel-demo/system.yaml
   kubectl -n kamerton port-forward svc/kamerton 8080
   ```

   The UI is at http://localhost:8080. Set up Telegram or email under
   «Каналы» to receive notifications.

4. Break something. The shop has fault flags; open its flag page and turn
   one on, for example `paymentFailure`:

   ```sh
   kubectl -n otel-demo port-forward svc/frontend-proxy 8081:8080
   ```

   Flags are at http://localhost:8081/feature. Within a few minutes the
   alert appears under «Оповещения» and in the configured channel.

## Second Prometheus

```sh
kubectl apply -f second-prometheus/prometheus.yaml
```

Then in Kamerton open «Состояние Kamerton», press «Найти установленные»,
tick the new Prometheus and create a set from it. A system that picks this
set gets its rules labelled so that only this Prometheus loads them.
`kubectl delete namespace team-b` removes it.

## По-русски

Здесь лежат готовые наборы файлов, чтобы попробовать Kamerton: демо-магазин
OpenTelemetry Demo с включаемыми сбоями и второй Prometheus для проверки
нескольких наборов программ. Шаги выше: собрать образ и поставить Kamerton,
поставить магазин, применить описание магазина, включить сбой на странице
флагов и увидеть оповещение.

## Licence

AGPL-3.0-only, see [LICENSE](LICENSE), the same licence as Kamerton.
