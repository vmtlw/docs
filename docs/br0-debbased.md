---
title: br0 in debian
icon: lucide/file-terminal
---


## Создание bridge с помощью NetworkManager (nmcli)
### Просмотр существующих подключений:
```
nmcli connection show
```
---
title: br0 in Debian
icon: lucide/file-terminal
---
## Создание bridge с помощью NetworkManager (nmcli)
### Просмотр существующих подключений
```bash
nmcli connection show
```
### Создание bridge-интерфейса
```bash
nmcli connection add type bridge ifname br0 con-name bridge-br0
```
note con-name лучше задавать явно, чтобы не полагаться на автогенерацию имени.

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
Важно: у физического интерфейса не должно оставаться отдельного DHCP-подключения — IP должен получать только br0.

### Проверка
```bash
ip addr show br0
bridge link
```
## Создание bridge с помощью netplan (systemd-networkd)
Важно
Netplan применяется только в системах с systemd-networkd.
Для Debian это имеет смысл, если вы не используете NetworkManager.

### Подготовка системы
```bash
apt purge network-manager ifupdown resolvconf
apt install netplan.io systemd-resolved
systemctl enable systemd-networkd systemd-resolved
```
networking.service удалять не обязательно — он не используется без ifupdown.

### Пример netplan-конфига
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
### Применение
```bash
netplan apply
```
Проверка
```bash
networkctl status br0
ip addr show br0
bridge link
```
Пояснение параметров bridge
```yaml
parameters:
  stp: false # отключает Spanning Tree Protocol

  forward-delay: 0 # убирает задержку перед началом передачи трафика
```
### Рекомендации
VM / single-host / lab → отключать STP допустимо
Физическая сеть / несколько uplink'ов → STP лучше включить
При stp: false параметр forward-delay фактически не влияет на поведение, но его часто указывают для явности.
netplan + networkd — предпочтительно для серверов и IaC
