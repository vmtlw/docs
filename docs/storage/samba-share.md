---
title: Настройка Samba
icon: lucide/folder-sync
tags:
  - storage
  - network
  - file sharing
---

# Настройка Samba для общего доступа к файлам

[Samba](https://www.samba.org/) - это свободное программное обеспечение, которое реализует протокол SMB/CIFS для обмена файлами между Linux и Windows системами.

## Базовая конфигурация

Ниже приведен пример конфигурации с двумя общими папками - одной публичной и одной приватной, с поддержкой символических ссылок Unix.

```bash title="/etc/samba/smb.conf"
[global]
# Разрешить небезопасные широкие ссылки
allow insecure wide links = yes
# Использовать sendfile для повышения производительности
use sendfile = yes
# Закрывать неактивные соединения через 30 минут
deadtime = 30
# Включить поддержку многоканального режима для увеличения пропускной способности
server multi channel support = yes

[downloads]
# Путь к каталогу, который нужно сделать общедоступным
path = /mnt/sda1/downloads
# Публичный доступ без пароля
public = yes
# Разрешить запись
writable = yes
# Принудительно установить режим создания файлов 0666 (чтение и запись для всех)
force create mode = 0666
# Принудительно установить режим создания каталогов 0777 (полный доступ для всех)
force directory mode = 0777
# Показывать в списке общих папок
browseable = yes
# Следовать по символическим ссылкам
follow symlinks = yes
# Разрешить использование широких ссылок (ссылок вне шары)
wide links = yes

[films]
path = /mnt/sda1/films
public = yes
writable = yes
force create mode = 0666
force directory mode = 0777
browseable = yes
follow symlinks = yes
wide links = yes
```

## Установка Samba

Перед настройкой необходимо установить Samba:

```bash
# Для Debian/Ubuntu
sudo apt install samba

# Для CentOS/RHEL
sudo yum install samba

# Для Arch Linux
sudo pacman -S samba
```

## Управление службой

```bash
# Запуск службы
sudo systemctl start smbd

# Включение автозапуска
sudo systemctl enable smbd

# Перезапуск после изменения конфигурации
sudo systemctl restart smbd

# Проверка статуса
sudo systemctl status smbd
```

## Безопасность

!!! warning "Безопасность"
    Использование публичных шар без пароля создает потенциальную угрозу безопасности. В продакшн-окружении рекомендуется настроить аутентификацию и ограничить доступ.

### Настройка защищенной шары

```bash
[secure]
path = /mnt/secure
public = no
writable = yes
valid users = @sambausers
create mask = 0660
directory mask = 0770
browseable = yes
```

### Управление пользователями

```bash
# Добавление пользователя Samba
sudo smbpasswd -a username

# Включение пользователя
sudo smbpasswd -e username

# Отключение пользователя
sudo smbpasswd -d username

# Удаление пользователя
sudo smbpasswd -x username
```

## Подключение к шаре

### Из Linux

```bash
# Подключение шары
sudo mount -t cifs //server_ip/share_name /mnt/mountpoint -o username=user,password=pass

# Постоянное подключение через fstab
# Добавьте в /etc/fstab:
//server_ip/share_name /mnt/mountpoint cifs username=user,password=pass,uid=local_user,gid=local_group 0 0
```

### Из Windows

1. Откройте Проводник
2. В адресной строке введите `\\server_ip\share_name`
3. Если требуется, введите имя пользователя и пароль
