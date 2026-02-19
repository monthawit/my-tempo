# my-tempo
Grafana Tempo

## Create Basic Authen user pass base64

```bash
# ตัวอย่าง: สร้างจาก username 'admin' และ password 'password123'
echo -n "admin:password123" | base64
# สมมติได้ค่า: YWRtaW46cGFzc3dvcmQxMjM=

นำไปใช้ใน Headers สำหรับ Tenant และ Auth
    headers:
      # แทนที่ด้วย Tenant ID ที่คุณตั้งไว้ใน Tempo
      X-Scope-OrgID: "my-tenant-id"
      # แทนที่ด้วยค่า Base64 ที่ได้จากขั้นตอนข้างบน
      Authorization: "Basic YWRtaW46cGFzc3dvcmQxMjM="
```
## Create Basic Authen Secret  Username password base64 
```bash
echo -n "admin:pass" | base64

xxxxxxxxxx1234ABC==

kubectl create secret generic tempo-auth \
--from-literal=auth-header="X-Scope-OrgID=otel-demo,Authorization=Basic xxxxxxxxxx1234ABC==" \
-n otel-demo --dry-run=client -o yaml | kubectl apply -f -

```

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

## Deployment ENV

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-db-app
  namespace: otel-demo
spec:
  template:
    metadata:
      labels:
        app: python-db-app
      annotations:
        # ส่วนสำคัญ: บอกให้ OTel Operator ฉีดเครื่องมือเข้าไป
        instrumentation.opentelemetry.io/inject-python: "true"
        # กรณีมีหลาย instrumentation  
        instrumentation.opentelemetry.io/otel-python-instrumentation: "otel-demo/my-instrumentation" 
    spec:
      containers:
      - name: app
        env:
        - name: OTEL_SERVICE_NAME
          value: python-db-app-01

## for grafana
        - name: GF_TRACING_OPENTELEMETRY_OTLP_ADDRESS
          value: "https://tempo-gateway.traces.olsxops.com/v1/traces"
        - name: GF_TRACING_OPENTELEMETRY_OTLP_PROTOCOL
          value: "http" # หรือ grpc ขึ้นอยู่กับ gateway คุณ
        - name: GF_TRACING_OPENTELEMETRY_OTLP_HEADERS
          value: "X-Scope-OrgID=otel-demo,Authorization=Basic xxxxxx=="           
        - name: GF_TRACING_OPENTELEMETRY_OTLP_SAMPLING_FRACTION
          value: "1.0"            
        - name: OTEL_SERVICE_NAME
          value: "grafana-web"
        - name: GF_TRACING_OPENTELEMETRY_OTLP_SERVICE_NAME
          value: "grafana-web"
```
