---
title: Kafka
icon: lucide/cpu
---

### посмотреть количесво сообщений в топиках - последнее число 
``` bash
kafka-run-class kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic notification-enricher
```

### Посмотреть все сообщения не учитывая смещение
```

kafka-console-consumer --bootstrap-server localhost:9092 --topic rule-engine-output-events --from-beginning
```

### Посмотреть какой лаг у консюмера 
```
kafka-consumer-groups --bootstrap-server localhost:9092 --group notification-enricher --describe
```

