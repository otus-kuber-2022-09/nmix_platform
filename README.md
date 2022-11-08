# nmix_platform

## 2. k8s_intro_hw

* установка kubectl и minikube
* разработан Dockerfile для web-сервера, образ выгружен в публичное хранилище
* составлен манифест, в minikube запущен pod с образом web-сервера
* отрабатоны основные команды kubectl - get, describe, delete, logs, run
* с использованием команды kubectl run получен манифест pod-а frontend
* выполнена отладка pod-а frontend, манифест скорректирован

## 3. kubernetes-controllers

### Выполнено

* запуск kubernetes-кластера через kind
* создание и управление ReplicaSet
* создание и управление Deployment
* создание и управление DaemonSet

### Как запустить проект

* kubectl apply -f kubernetes-controllers/frontend-replicaset.yaml
* kubectl apply -f kubernetes-controllers/frontend-deployment.yaml
* etc

### Как проверить работоспособность

* kubectl get pods
* kubectl get replicaset
* kubectl get deployments
* kubectl get daemonsets

## 4. kubernetes-networks

### Выполнено

* добавлены readinessProbe и livenessProbe в pod
* создан сервис ClusterIP, активирован IPVS
* установлен MetalLB, создан сервис LoadBalancer, получен доступ к приложению в поде через EXTERNAL IP
* установлен Ingress, создан ресурс Ingress и Headless сервис, ингрес настроен на сервис

### Как запустить проект

* устанавливаем MetalLB и Ingress
* kubectl apply -f kubernetes-networks/metallb-config.yaml
* kubectl apply -f kubernetes-networks/nginx-lb.yaml
* kubectl apply -f kubernetes-networks/web-svc-headless.yaml

### Как проверить работоспособность

* kubectl get ingress web
* curl http://INGRESS_IP/web


## 5. kubernetes-volumes

### Выполнено

* создан ресурс StatefulSet c PV и PVC для Minio
* создан Headless сервис для StatefulSet
* проверено взаимодействие с Minio c использованием mc
* создан ресурс Secret для секретов из StatefulSet

### Как запустить проект

Без секретов

```bash
kubectl apply -f kubernetes-volumes/minio-statefulset.yaml
kubectl apply -f kubernetes-volumes/minio-headless-service.yaml
```

С секретами

```bash
kubectl create secret generic minio \
  --from-literal=MINIO_ACCESS_KEY=minio \
  --from-literal=MINIO_SECRET_KEY=minio123

kubectl apply -f kubernetes-volumes/minio-statefulset-secrets.yaml
kubectl apply -f kubernetes-volumes/minio-headless-service.yaml
```

### Как проверить работоспособность

```bash
kubectl run mc --image minio/mc --command sleep 99999
kubectl exec mc -- mc alias set minio http://minio-0.minio:9000 minio minio123
kubectl exec mc -- mc cp /etc/resolv.conf minio:/1/resolv.conf
kubectl exec mc -- mc ls minio:/1
```

## 6. kubernetes-security

### Выполнено

 созданы сервисные аккаунты с доступом и без доступа к кластеру
- создано пространство имен, все сервисные аккаунты которого имеют доступ ко всем подам кластера
- создано пространство имен с ролями администратор и "просмотрщик" только для этого пространства имен.

### Как запустить проект:
 - применить все манифесты из директории kubernetes-security

### Как проверить работоспособность:
 - тесты должны пройти

## 7. kubernetes-templating

### Выполнено

- установлен nginx-ingress через helm
 - установлен cert-manager через helm
 - установлены chartmuseum, harbor через helm
 - установлен hipster-shop через helm
 - выделен сервис frontend и установлен как отдельная схема helm
 - выделены сервисы сервисы paymentservice и shippingservice и установлены через kubecfg
 - выделен сервис currencyservice и установлен через kustomize

### Как запустить проект

```bash
# --- nginx-ingress
kubectl create ns nginx-ingress
helm repo add stable https://charts.helm.sh/stable
helm upgrade --install nginx-ingress nginx-stable/nginx-ingress --wait --namespace=nginx-ingress --version=0.15.1

# --- cert-manager
helm repo add jetstack https://charts.jetstack.io
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.16.1/cert-manager.crds.yaml
helm upgrade --install cert-manager kubernetes-templating/jetstack/cert-manager \
  --wait \
  --namespace=cert-manager \
  --version=0.16.1

# --- chartmuseum
kubectl create ns charmuseum
helm repo add stable https://charts.helm.sh/stable
helm upgrade --install chartmuseum stable/chartmuseum --wait \
  --namespace=chartmuseum \
  --version=2.13.2 \
  -f kubernetes-templating/chartmuseum/values.yaml

# --- harbor
kubectl create ns harbor
helm repo add harbor https://helm.goharbor.io
helm upgrade --install harbor harbor/harbor --wait \
  --namespace=harbor \
  --version=1.1.2 \
  -f harbor/values.yaml

# --- frontend
kubecfg update services.jsonnet --namespace hipster-shop

# --- kustomize
kubectl apply -k kubernetes-templating/kustomize/overrides/hipster-shop
kubectl apply -k kubernetes-templating/kustomize/overrides/hipster-shop-prod
```

### Как проверить работоспособность

> Ссылки доступны после запуска кластера по запросу

 - перейти по ссылке https://chartmuseum.otus.onrails.ru
 - перейти по ссылке https://harbor.otus.onrails.ru
 - перейти по ссылке https://shop.otus.onrails.ru


## 8. kubernetes-logging

### Выполнено

* запущен кластер в Яндекс.Облако
* установлен nginx-ingress
* установлены elastic, kibana и fluent-bit
* в kibana построена визуализация логов nginx-ingress
* установлен prometheus
* метрики elasticsearch прокинуты в prometheus через elasticsearch-exporter
* установлены loki и promtail
* логи nginx-ingress прокинуты в loki, доступны через grafana
* метрики nginx-ingress прокинуты в prometheus через serviceMonitor
* в grafana построены панели для nginx-ingress


### Как запустить проект

```bash
# --- https://github.com/helm/charts/tree/master/stable/nginx-ingress
helm repo add nginx-stable https://helm.nginx.com/stable 
helm upgrade --install nginx-ingress stable/nginx-ingress --wait \
  --namespace=nginx-ingress \
  --version=1.41.3 \
  -f nginx-ingress.values.yaml

kubectl create ns observability
helm upgrade --install elasticsearch elastic/elasticsearch --namespace observability
helm upgrade --install kibana elastic/kibana --namespace observability
helm upgrade --install fluent-bit stable/fluent-bit --namespace observability

# quick start for prometheus-operator
# https://prometheus-operator.dev/docs/prologue/quick-start/

# elasticsearch-exporter source https://github.com/prometheus-community/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm upgrade --install elasticsearch-exporter prometheus-community/prometheus-elasticsearch-exporter \
  --set es.uri=http://elasticsearch-master:9200 \
  --set serviceMonitor.enabled=true \
  --namespace=observability

# --- https://github.com/grafana/loki/blob/v1.3.0/docs/installation/helm.md
helm repo add loki https://grafana.github.io/loki/charts
helm repo update
helm upgrade --install loki loki/loki -n observability
helm upgrade --install promtail loki/promtail --set "loki.serviceName=loki" -n observability

# --- https://github.com/grafana/loki/blob/v1.3.0/docs/clients/promtail/installation.md
helm upgrade --install promtail loki/promtail --set "loki.serviceName=loki" -n observability

# --- https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  -n observability -f prometheus-operator.values.yaml
```

### Как проверить работоспособность

- прокидываем порты kibana, импортируем визуализацию
- прокидываем порты grafana, наблюдаем метрики и логи
