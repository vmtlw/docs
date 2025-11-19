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