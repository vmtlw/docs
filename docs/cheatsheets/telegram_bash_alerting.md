---
title: Оповещения в Telegram из Bash
icon: lucide/bell
---

# Отправка оповещений в Telegram из Bash

Этот скрипт поможет быстро отправить в Telegram любые данные, переданные ему в stdin.

## Скрипт для отправки

```bash
#!/bin/bash
API_TOKEN="111111111111:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
CHAT_ID="999999999"

curl -s -X POST \
  -d chat_id=$CHAT_ID \
  -d text="$(cat)" \
  https://api.telegram.org/bot$API_TOKEN/sendMessage
```

## Примеры использования

Отправка простого текстового сообщения:

```bash
echo "Hello World!" | ./alert.sh
```

Отправка результата выполнения команды:

```bash
df -h | ./alert.sh
```

Отправка уведомления о завершении длительного процесса:

```bash
long_running_command && echo "Процесс завершен успешно!" | ./alert.sh
```

## Настройка

1. Создайте бота через @BotFather в Telegram и получите API_TOKEN
2. Узнайте CHAT_ID вашего чата или группы
3. Замените значения в скрипте и сохраните его как alert.sh
4. Сделайте скрипт исполняемым: `chmod +x alert.sh`
