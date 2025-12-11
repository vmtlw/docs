---
title: telegram alerting
icon: lucide/globe-lock
---

#### этот скрипт поможет что угодно переданное ему в stdin быстро отправить в телеграм 

#!/bin/bash file="./script.sh"

API_TOKEN="111111111111:aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
CHAT_ID="999999999"

curl -s -X POST \
-d chat_id=$CHAT_ID \
-d text="`cat`" \
https://api.telegram.org/bot$API_TOKEN/sendMessage
```
Пример использования:
```bash
echo "Hello World!" | ./alert.sh
```
