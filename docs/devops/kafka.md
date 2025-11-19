---
title: Работа с Apache Kafka
icon: lucide/cpu
---

# Apache Kafka

Apache Kafka — это распределенная платформа потоковой передачи данных, используемая для создания конвейеров данных в реальном времени и приложений для потоковой обработки. 

## Основные концепции

- **Брокеры (Brokers)**: Серверы, составляющие кластер Kafka.
- **Топики (Topics)**: Категории для организации сообщений.
- **Партиции (Partitions)**: Части топика, распределённые по брокерам.
- **Продюсеры (Producers)**: Приложения, которые отправляют сообщения в Kafka.
- **Консьюмеры (Consumers)**: Приложения, которые считывают сообщения из Kafka.
- **Группы потребителей (Consumer Groups)**: Группы консьюмеров, которые совместно обрабатывают сообщения.
- **Смещение (Offset)**: Позиция сообщения в партиции.

## Команды для работы с топиками

### Создание топика

```bash
kafka-topics --bootstrap-server localhost:9092 --create --topic my-topic --partitions 3 --replication-factor 1
```

### Просмотр списка топиков

```bash
kafka-topics --bootstrap-server localhost:9092 --list
```

### Просмотр информации о топике

```bash
kafka-topics --bootstrap-server localhost:9092 --describe --topic my-topic
```

### Посмотреть количество сообщений в топике (последнее число)

```bash
kafka-run-class kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic notification-enricher
```

## Работа с консьюмерами

### Просмотр всех сообщений с начала

```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic rule-engine-output-events --from-beginning
```

### Просмотр лага консьюмера

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --group notification-enricher --describe
```

### Создание консьюмера с указанием группы

```bash
kafka-console-consumer --bootstrap-server localhost:9092 --topic my-topic --group my-group
```

## Работа с продюсерами

### Отправка сообщений через консольный продюсер

```bash
kafka-console-producer --bootstrap-server localhost:9092 --topic my-topic
```

После запуска команды можно вводить сообщения построчно и отправлять их, нажимая Enter.

### Отправка сообщений с ключом

```bash
kafka-console-producer --bootstrap-server localhost:9092 --topic my-topic --property "parse.key=true" --property "key.separator=:"
```

Пример ввода: `key1:value1`

## Мониторинг и администрирование

### Просмотр групп консьюмеров

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --list
```

### Сброс смещения консьюмера

```bash
kafka-consumer-groups --bootstrap-server localhost:9092 --group my-group --topic my-topic --reset-offsets --to-earliest --execute
```

### Изменение конфигурации топика

```bash
kafka-configs --bootstrap-server localhost:9092 --entity-type topics --entity-name my-topic --alter --add-config retention.ms=86400000
```

## Примеры использования в продакшн

### Запуск многоузлового кластера

```bash
# Запуск ZooKeeper (для Kafka < 3.0)
zookeeper-server-start /path/to/zookeeper.properties

# Запуск брокера Kafka
kafka-server-start /path/to/server-1.properties
```

### Типичная конфигурация высокодоступного кластера

- Минимум 3 брокера для высокой доступности
- Фактор репликации 3 для важных топиков
- Включение автоматического создания топиков: `auto.create.topics.enable=true`
- Настройка удержания данных: `log.retention.hours=168` (1 неделя)

## Решение распространенных проблем

### Лаги консьюмеров

Если наблюдаются большие лаги консьюмеров:
1. Увеличьте количество партиций топика
2. Увеличьте количество консьюмеров в группе
3. Оптимизируйте обработку сообщений в консьюмере

### Проблемы с производительностью

Для повышения производительности:
1. Увеличьте `batch.size` у продюсера
2. Настройте `linger.ms` для группировки сообщений
3. Увеличьте `buffer.memory` для накопления сообщений

### Проблемы с дисковым пространством

Если заканчивается дисковое пространство:
1. Уменьшите время хранения сообщений: `log.retention.hours`
2. Ограничьте размер хранимых данных: `log.retention.bytes`
3. Добавьте больше дискового пространства к брокерам
