# nmix_platform

## 1. k8s_intro_hw

* установка kubectl и minikube
* разработан Dockerfile для web-сервера, образ выгружен в публичное хранилище
* составлен манифест, в minikube запущен pod с образом web-сервера
* отрабатоны основные команды kubectl - get, describe, delete, logs, run
* с использованием команды kubectl run получен манифест pod-а frontend
* выполнена отладка pod-а frontend, манифест скорректирован

## 2. kubernetes-controllers

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

## 3. kubernetes-networks


