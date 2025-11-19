---
title: S3 хранилище
icon: lucide/cloud
tags:
  - storage
  - cloud
---

# Работа с S3-совместимыми хранилищами

[S3 (Simple Storage Service)](https://aws.amazon.com/s3/) - это объектное хранилище, изначально разработанное Amazon. Сейчас существует множество S3-совместимых сервисов и решений, включая MinIO, Ceph, и другие.

## Использование MinIO Client (mc)

MinIO Client (mc) - это современная альтернатива для unix-команд ls, cp, mkdir и других, которая поддерживает файловые системы и облачные хранилища, совместимые с S3 API.

### Установка MinIO Client

```bash
# Для Linux x86_64
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
sudo mv mc /usr/local/bin/mc
```

### Настройка подключения к S3

Создайте алиас для вашего S3-совместимого сервера:

```bash
mc alias set myminio http://127.0.0.1:9000 $USER $PASSWORD
```

Где:
- `myminio` - это имя алиаса
- `http://127.0.0.1:9000` - адрес S3-сервера
- `$USER` - имя пользователя
- `$PASSWORD` - пароль

### Создание бакета (корзины)

Для создания нового бакета используйте команду:

```bash
mc mb myminio/my-bucket
```

### Базовые операции с файлами

```bash
# Загрузить файл в бакет
mc cp myfile.txt myminio/my-bucket/

# Скачать файл из бакета
mc cp myminio/my-bucket/myfile.txt .

# Список объектов в бакете
mc ls myminio/my-bucket/

# Удаление объекта
mc rm myminio/my-bucket/myfile.txt
```

!!! tip "Совет"
    Для интеграции с S3 в ваших приложениях можно использовать библиотеки для различных языков программирования, такие как boto3 для Python, AWS SDK для JavaScript и т.д.
