# my-tempo
Grafana Tempo

## Install Opentelemetry Operator 

```bash
kubectl create ns opentelemetry


helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

helm install opentelemetry-operator open-telemetry/opentelemetry-operator \
  --set admissionWebhooks.certManager.enabled=false -n opentelemetry

```
## Install Opentelemetry Collector 

```bash
================ opentelemetry collector ===============

helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts

helm install my-opentelemetry-collector open-telemetry/opentelemetry-collector \
   --set image.repository="otel/opentelemetry-collector-k8s" \
   --set mode=<daemonset|deployment|statefulset>


================ opentelemetry collector + values.yaml ===============

helm upgrade --install my-opentelemetry-collector open-telemetry/opentelemetry-collector -f otel-collector-values.yaml -n  opentelemetry
```


