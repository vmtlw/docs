---
title: Longhorn
icon: lucide/database
tags:
  - kubernetes
  - storage
  - cloud-native
---

# Longhorn - распределенное хранилище для Kubernetes

[Longhorn](https://longhorn.io/) - это легковесная, надежная и мощная система распределенного блочного хранения с открытым исходным кодом для Kubernetes.

## Подготовка узлов

Перед установкой Longhorn необходимо подготовить каждый узел кластера Kubernetes. Выполните следующие команды на каждой ноде:

```bash
# Установка необходимых зависимостей
apt install -y nfs-common cryptsetup dmsetup open-iscsi

# Загрузка необходимых модулей ядра
modprobe -a iscsi_tcp dm_crypt

# Настройка автоматической загрузки модулей при старте системы
echo -e "iscsi_tcp\ndm_crypt" > /etc/modules-load.d/longhorn.conf

# Включение и запуск сервиса iSCSI
systemctl enable --now iscsid.service
```

## Установка Longhorn

### Установка с помощью Helm

```bash
# Добавление репозитория Helm
helm repo add longhorn https://charts.longhorn.io
helm repo update

# Установка Longhorn
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace
```

### Установка с помощью kubectl

```bash
# Применение манифестов
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
```

## Доступ к UI Longhorn

После установки вы можете получить доступ к пользовательскому интерфейсу Longhorn через:

```bash
# Создание прокси для доступа к UI
kubectl -n longhorn-system port-forward svc/longhorn-frontend 8000:80
```

Затем откройте браузер и перейдите по адресу http://localhost:8000

## Настройка StorageClass

Для использования Longhorn в качестве стандартного хранилища в Kubernetes создайте и примените следующий манифест:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: longhorn
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: driver.longhorn.io
allowVolumeExpansion: true
parameters:
  numberOfReplicas: "3"
  staleReplicaTimeout: "30"
  fromBackup: ""
```

## Основные операции

### Создание PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-test-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 2Gi
```

### Создание резервной копии

Резервное копирование можно настроить через пользовательский интерфейс Longhorn или с помощью API.

!!! warning "Важно"
    Перед обновлением кластера или критическими изменениями всегда делайте резервные копии ваших данных.
