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

```bash
kubectl create secret generic minio \
  --from-literal=MINIO_ACCESS_KEY=minio \
  --from-literal=MINIO_SECRET_KEY=minio123

kubectl apply -f kubernetes-volumes/minio-statefulset.yaml
kubectl apply -f kubernetes-volumes/minio-headless-service.yaml
```

### Как проверить работоспособность

```bash
kubectl run mc --image minio/mc --command sleep 99999
kubectl exec mc -- mc alias set minio http://minio-0.minio:9000 minio minio123
kubectl exec mc -- mc cp /etc/resolv.conf minio:/1/resolv.conf
kubectl exec mc -- mc ls minio:/1
```
