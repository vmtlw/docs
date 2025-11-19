---
title: Настройка Bridge в Debian
icon: lucide/git-branch-plus
tags:
  - networking
  - debian
  - virtualization
---

# Настройка сетевого моста (Bridge) в Debian

![Bridge Network](../assets/bridge-network.png){ align=right width="300" }

Сетевой мост (bridge) - это устройство, которое объединяет несколько сегментов сети в единую сеть на канальном уровне. Это полезно для виртуализации, контейнеров и других задач, требующих прозрачного сетевого взаимодействия.

## Создание bridge с помощью NetworkManager (nmcli)

Этот метод подходит для десктопных систем и серверов, где установлен NetworkManager.

### Просмотр существующих подключений

```bash
nmcli connection show
```

### Создание bridge-интерфейса

```bash
nmcli connection add type bridge ifname br0 con-name bridge-br0
```

!!! note
    Параметр con-name лучше задавать явно, чтобы не полагаться на автогенерацию имени.

### Добавление физического интерфейса в bridge

```bash
nmcli connection add type bridge-slave ifname enp8s0 master bridge-br0
```

### Включение bridge

```bash
nmcli connection up bridge-br0
```

### Отключение старого подключения к физическому интерфейсу

```bash
nmcli connection down "Wired connection 1"
```

!!! warning "Важно"
    У физического интерфейса не должно оставаться отдельного DHCP-подключения — IP должен получать только br0.

### Проверка работоспособности

```bash
ip addr show br0
bridge link
```

## Создание bridge с помощью netplan (systemd-networkd)

Netplan применяется в системах с systemd-networkd. Для Debian это имеет смысл, если вы не используете NetworkManager.

### Подготовка системы

```bash
apt purge network-manager ifupdown resolvconf networking
apt install netplan.io systemd-resolved
systemctl enable --now systemd-networkd systemd-resolved
```

!!! note
    Удалять службу networking.service не обязательно — она не используется без ifupdown.

### Пример netplan-конфига

Создайте файл `/etc/netplan/01-bridge.yaml` со следующим содержимым:

```yaml
network:
  version: 2
  renderer: networkd

  ethernets:
    eno1:
      dhcp4: false

  bridges:
    br0:
      interfaces:
        - eno1
      dhcp4: true
      nameservers:
        search:
          - vmtlw.ru
      parameters:
        stp: false
        forward-delay: 0
```

### Применение конфигурации

```bash
netplan apply
```

### Проверка работоспособности

```bash
networkctl status br0
ip addr show br0
bridge link
```

## Параметры bridge и рекомендации

### Пояснение параметров

```yaml
parameters:
  stp: false    # отключает Spanning Tree Protocol
  forward-delay: 0    # убирает задержку перед началом передачи трафика
```

### Рекомендации по настройке

- **Виртуальные машины / одиночный хост / лабораторная среда**: отключение STP допустимо
- **Физическая сеть / несколько uplink'ов**: STP лучше включить
- При `stp: false` параметр `forward-delay` фактически не влияет на поведение, но его часто указывают для явности
- netplan + networkd — предпочтительный подход для серверов и инфраструктуры как код (IaC)

## Дополнительные возможности

### Фиксированный IP-адрес вместо DHCP

```yaml
bridges:
  br0:
    interfaces:
      - eno1
    addresses:
      - 192.168.1.10/24
    routes:
      - to: default
        via: 192.168.1.1
    nameservers:
      addresses:
        - 8.8.8.8
        - 8.8.4.4
      search:
        - vmtlw.ru
```
