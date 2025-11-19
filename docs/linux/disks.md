---
icon: lucide/hard-drive
title: Управление дисками в Linux
---

# Управление дисками в Linux

В этом руководстве собраны команды и рекомендации по работе с дисками и разделами в Linux.

## Информация о дисках

### Просмотр списка дисков и разделов

```bash
# Просмотр всех блочных устройств
lsblk

# Подробная информация о дисках
fdisk -l

# Просмотр UUID и меток разделов
blkid

# Информация об использовании дискового пространства
df -h
```

### Проверка состояния и здоровья диска

```bash
# Проверка SMART-статуса
smartctl -a /dev/sda

# Проверка на битые сектора
badblocks -v /dev/sda

# Тестирование производительности чтения
hdparm -t /dev/sda

# Тестирование производительности записи
dd if=/dev/zero of=/tmp/test bs=1G count=1 oflag=dsync
```

## Работа с разделами

### Создание таблицы разделов и разделов

```bash
# Запуск fdisk в интерактивном режиме
fdisk /dev/sda

# Альтернативно можно использовать parted
parted /dev/sda
```

### Форматирование разделов

```bash
# Форматирование в ext4
mkfs.ext4 /dev/sda1

# Форматирование в XFS
mkfs.xfs /dev/sda2

# Создание раздела подкачки
mkswap /dev/sda3
swapon /dev/sda3
```

### Монтирование разделов

```bash
# Ручное монтирование
mount /dev/sda1 /mnt/data

# Добавление записи в /etc/fstab для автоматического монтирования
echo "UUID=$(blkid -s UUID -o value /dev/sda1) /mnt/data ext4 defaults 0 2" >> /etc/fstab
```

## Управление дисками

### Отключение диска 

```bash
# Отключение питания диска через udisks
udisksctl power-off -b /dev/sdb 

# Альтернативный способ через hdparm
hdparm -s 0 /dev/sda
```

### Перемещение данных между дисками

```bash
# Клонирование диска
dd if=/dev/sda of=/dev/sdb bs=4M status=progress

# Копирование с сохранением прав доступа
rsync -av /source/ /destination/
```

## Логические тома (LVM)

### Создание LVM-томов

```bash
# Создание физического тома
pvcreate /dev/sda1

# Создание группы томов
vgcreate vg_data /dev/sda1 /dev/sdb1

# Создание логического тома
lvcreate -L 10G -n lv_data vg_data
```

### Изменение размера LVM-томов

```bash
# Увеличение размера логического тома
lvextend -L +5G /dev/vg_data/lv_data
resize2fs /dev/vg_data/lv_data

# Уменьшение размера логического тома
# Сначала нужно отмонтировать файловую систему
umount /dev/vg_data/lv_data
resize2fs /dev/vg_data/lv_data 5G
lvreduce -L 5G /dev/vg_data/lv_data
```

## RAID-массивы

### Создание программного RAID-массива

```bash
# Создание RAID1 (зеркало)
mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1

# Создание RAID5
mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda1 /dev/sdb1 /dev/sdc1
```

### Мониторинг RAID-массива

```bash
# Просмотр состояния всех массивов
cat /proc/mdstat

# Подробная информация о RAID-массиве
mdadm --detail /dev/md0
```

## Шифрование дисков

### Шифрование с помощью LUKS

```bash
# Шифрование раздела
cryptsetup luksFormat /dev/sda1

# Открытие зашифрованного раздела
cryptsetup luksOpen /dev/sda1 encrypted_data

# Закрытие зашифрованного раздела
cryptsetup luksClose encrypted_data
```

## Решение проблем

### Восстановление данных

```bash
# Копирование с игнорированием ошибок
dd if=/dev/sda of=/dev/sdb bs=4M conv=noerror,sync status=progress

# Восстановление удалённых файлов
testdisk /dev/sda
```

### Проверка и исправление ошибок

```bash
# Проверка и исправление ошибок в файловой системе ext4
e2fsck -f /dev/sda1

# Проверка и исправление ошибок в файловой системе XFS
xfs_repair /dev/sda2
```
