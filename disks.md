---
hide_title: true
title:  Отключение диска
---

### Manage disks

Для отключения диска выполните:
 
```
udisksctl power-off -b /dev/sdb 

# или:

hdparm -s 0 /dev/sda
```
