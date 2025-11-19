---
title: br0 in debian
icon: lucide/file-terminal
---

Отобразить список подключений:

```
nmcli con show
```

Создайте бридж:
```
nmcli con add type bridge ifname br0
```

Создайте подчиненный интерфейс:
```
nmcli con add type bridge-slave ifname enp8s0 master bridge-br0
```

Запустите бридж:
```
nmcli con up bridge-br0
```

Отключите остальные сети:
```
nmcli con down "Wired connection 1"
```
