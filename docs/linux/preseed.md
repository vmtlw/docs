---
icon: lucide/file-terminal
title: Автоматическая установка Debian с Preseed
---

# Автоматическая установка Debian с помощью Preseed

Preseed - это метод автоматизации установки Debian и Ubuntu, позволяющий полностью автоматизировать процесс установки операционной системы без взаимодействия с пользователем.

## Что такое Preseed?

Preseed-файл содержит ответы на вопросы, которые задаёт установщик Debian (debian-installer) в процессе установки. Это позволяет полностью автоматизировать процесс и развертывать системы с идентичной конфигурацией.

## Пример файла preseed.cfg

Ниже представлен пример конфигурационного файла для автоматической установки Debian с русской локализацией, использованием XFS и настройкой EFI-загрузчика:

```bash
### Локализация
d-i debian-installer/language string ru
d-i debian-installer/country string RU
d-i debian-installer/locale string ru_RU.UTF-8
d-i keyboard-configuration/xkb-keymap select en

### Настройка сети
d-i netcfg/choose_interface select auto
d-i netcfg/get_hostname string debian
d-i netcfg/get_domain string local

d-i partman-auto/method string regular
d-i partman-auto/disk string /dev/sda

d-i partman-auto/expert_recipe string \
    boot-efi-root-xfs :: \
        100 100 100 fat32 \
            $primary{ } $bootable{ } \
            method{ format } format{ } \
            use_filesystem{ } filesystem{ fat32 } \
            mountpoint{ /boot/efi } \
        . \
        50000 50000 50000 xfs \
            $primary{ } \
            method{ format } format{ } \
            use_filesystem{ } filesystem{ xfs } \
            mountpoint{ / } \
        .

### Разметка диска
d-i partman-auto/choose_recipe select boot-efi-root-xfs
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

### Настройка пользователя
d-i passwd/root-login boolean true
d-i passwd/make-user boolean false

### Выбор пакетов
tasksel tasksel/first multiselect minimal
d-i pkgsel/include string openssh-server
d-i pkgsel/upgrade select none
d-i pkgsel/install-recommends boolean false

### Настройка часового пояса
d-i time/zone string Europe/Moscow
d-i clock-setup/utc boolean true
d-i clock-setup/ntp boolean true

### Настройка GRUB
d-i grub-installer/only_debian boolean true
d-i grub-installer/with_other_os boolean false
d-i grub-installer/bootdev string default

### Завершение установки
d-i finish-install/reboot_in_progress note
```

## Как использовать Preseed

### Шаг 1: Создание файла preseed.cfg

1. Создайте файл `preseed.cfg` на основе приведенного выше примера
2. Настройте параметры согласно вашим требованиям (разделы диска, пакеты, пользователей и т.д.)
3. Сохраните файл как `preseed.cfg`

### Шаг 2: Подготовка установочного носителя

#### Вариант 1: Добавление preseed-файла на USB-накопитель

1. Создайте загрузочную флешку с образом Debian
2. Скопируйте файл `preseed.cfg` в корневой каталог флешки

#### Вариант 2: Встраивание preseed-файла в ISO-образ

```bash
# Распакуйте ISO-образ
mkdir -p /tmp/debian-iso
mount -o loop debian.iso /mnt
cp -rT /mnt /tmp/debian-iso
umount /mnt

# Добавьте preseed-файл
cp preseed.cfg /tmp/debian-iso/

# Создайте новый ISO-образ
xorriso -as mkisofs -r -J -joliet-long -l -b isolinux/isolinux.bin \
  -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 \
  -boot-info-table -eltorito-alt-boot -e boot/grub/efi.img \
  -no-emul-boot -isohybrid-gpt-basdat -isohybrid-apm-hfsplus \
  -o debian-preseed.iso /tmp/debian-iso/
```

### Шаг 3: Запуск установки с использованием preseed

#### Вариант 1: Указание preseed-файла при загрузке

При загрузке с установочного носителя нажмите Tab (для BIOS) или e (для UEFI) и добавьте следующие параметры к строке загрузки:

```
auto=true priority=critical file=/cdrom/preseed.cfg
```

#### Вариант 2: Указание preseed-файла по URL

```
auto=true priority=critical preseed/url=http://server/path/to/preseed.cfg
```

### Шаг 4: Проверка установки

После завершения автоматической установки система перезагрузится и будет готова к использованию с настройками, указанными в preseed-файле.

## Важные замечания

- Убедитесь, что путь к preseed-файлу указан правильно
- Для полностью автоматической установки используйте `priority=critical`
- Для отладки preseed-установки используйте параметр `DEBCONF_DEBUG=5`
- Пароли в preseed-файле должны быть зашифрованы или устанавливаться через late_command

## Дополнительные ресурсы

- [Официальная документация Debian по Preseed](https://www.debian.org/releases/stable/amd64/apb.html)
- [Примеры preseed-файлов](https://www.debian.org/releases/stable/example-preseed.txt)
- [Wiki Debian по автоматизации установки](https://wiki.debian.org/DebianInstaller/Preseed)
