# my-tempo
Grafana Tempo

## Create user pass base64

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
## Create Secret  Username password base64 
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


