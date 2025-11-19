---
title: Wake-on-LAN (WoL)
icon: lucide/ethernet-port
tags:
  - networking
  - hardware
  - remote management
---

# Wake-on-LAN (WoL)

**Wake-On-LAN** - технология, позволяющая включать компьютер дистанционно по сети путём отправки специального пакета данных.

## Требования

### Требования к включаемому (ведомому) компьютеру

- ATX источник питания
- Материнская плата с поддержкой Wake-On-LAN
- Сетевой адаптер с поддержкой Wake-On-LAN
- Известный MAC-адрес сетевого адаптера

### Требования к управляющему (ведущему) компьютеру

- Специальная программа, умеющая отсылать Magic Packet

## Принцип работы

Ведомый компьютер находится в дежурном режиме (*stand by*) и выдает питание на сетевой адаптер. Сетевой адаптер находится в режиме пониженного энергопотребления, просматривая все пакеты, приходящие на его MAC-адрес, но ничего не отвечая на них. Если один из них окажется Magic Packet, то сетевой адаптер выдаёт сигнал на включение питания компьютера.

## Настройка Wake-on-LAN

### Включение поддержки WoL в BIOS/UEFI

Включите поддержку WoL в BIOS на ведомом компьютере. Это может быть:
- Пункт меню с названием "Wake On LAN Enable"
- "Power On By PCIE"
- Другие похожие параметры

В некоторых случаях этот режим в BIOS не меняется, а материнская плата поддерживает его по умолчанию.

### Проверка поддержки WoL сетевой картой

Чтобы определить, поддерживает ли сетевая карта Wake-on-LAN, загрузите ведомый компьютер и выполните команду:

```bash
ethtool eth0  # eth0 замените на имя вашего интерфейса
```

Результат будет примерно таким:

```
Settings for eth0:
Supported ports: [ MII ]
Supported link modes:   10baseT/Half 10baseT/Full
                        100baseT/Half 100baseT/Full
                        1000baseT/Full
Supports auto-negotiation: Yes
Advertised link modes:  10baseT/Half 10baseT/Full
                        100baseT/Half 100baseT/Full
                        1000baseT/Full
Advertised auto-negotiation: Yes
Speed: 1000Mb/s
Duplex: Full
Port: MII
PHYAD: 1
Transceiver: external
Auto-negotiation: on
Supports Wake-on: g
Wake-on: d
Link detected: yes
```

Обратите внимание на строки:
- **Supports Wake-on**: показывает доступные режимы сетевого адаптера для пробуждения (`g` - пробуждение по Magic Packet)
- **Wake-on**: показывает текущий режим (`d` означает выключенный WoL)

### Включение режима WoL на сетевой карте

Для включения режима Wake-on-LAN используйте команду:

```bash
ethtool -s eth0 wol g  # eth0 замените на имя вашего интерфейса
```

Для выключения режима Wake-on-LAN:

```bash
ethtool -s eth0 wol d  # eth0 замените на имя вашего интерфейса
```

!!! warning "Важно"
    Сетевой адаптер может не сохранять настройки WoL при перезагрузке. В этом случае необходимо настраивать его при каждой загрузке системы.

### Автоматическое включение WoL при загрузке системы

Добавьте в `/etc/conf.d/net` (для Gentoo) или создайте соответствующий скрипт в системе инициализации следующие строки:

```bash
preup() {
   if ethtool $1 | grep "Supports Wake-on:" | grep g >/dev/null;
     then
       ethtool -s $1 wol g
     fi
}
```

Для систем с systemd можно создать сервис или использовать файл `/etc/networkd-dispatcher/routable.d/99-wol` со следующим содержимым:

```bash
#!/bin/bash
IFACE=$1
if ethtool $IFACE | grep -q "Supports Wake-on:.*g" && ethtool $IFACE | grep -q "Wake-on:.*d"; then
    ethtool -s $IFACE wol g
fi
```

Не забудьте сделать файл исполняемым:

```bash
chmod +x /etc/networkd-dispatcher/routable.d/99-wol
```

## Определение MAC-адреса

Для получения MAC-адреса сетевого адаптера на ведомом компьютере можно:

### Метод 1: Использовать команду на ведомом компьютере

```bash
ifconfig -a
```

Результат:
```
eth0     Link encap:Ethernet  HWaddr 01:02:03:04:05:06
         inet addr:192.168.1.2  Bcast:192.168.1.255  Mask:255.255.255.0
         inet6 addr: fe80::215:f2ff:fe6f:3487/64 Scope:Link
         UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
         RX packets:71495 errors:0 dropped:0 overruns:0 frame:0
         TX packets:76190 errors:0 dropped:0 overruns:0 carrier:0
         collisions:0 txqueuelen:1000
         RX bytes:23164212 (22.0 MiB)  TX bytes:7625016 (7.2 MiB)
         Interrupt:217 Base address:0xd400
```

Или в современных системах:

```bash
ip link
```

### Метод 2: Посмотреть ARP-кэш на ведущем компьютере

```bash
arp
```

Результат:
```
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.0.1                 ether   00:01:02:03:04:05   C                     eth0
10.0.0.2                 ether   06:07:08:09:0a:0b   C                     eth0
10.0.0.3                 ether   0c:0d:0e:0f:10:11   C                     eth0
```

### Метод 3: Сканирование сети

Для того, чтобы все компьютеры сети попали в ARP-кэш, можно воспользоваться утилитой nmap:

```bash
nmap -v -sP 10.0.0.0/24  # Замените на ваш диапазон адресов
```

## Отправка Magic Packet

Для пробуждения компьютера используйте утилиту `wol` на управляющем компьютере:

```bash
wol 01:02:03:04:05:06  # Замените на MAC-адрес ведомого компьютера
```

## Дополнительные советы

!!! info "Первое включение"
    При работе с технологией Wake-on-LAN следует учитывать, что не все компьютеры включаются сразу после подключения в электрическую сеть. Это связано с отсутствием процесса инициализации подачи питания на сетевую карту. Поэтому следует произвести одно предварительное включение вручную.

!!! tip "Автоматическое включение после сбоя питания"
    Если существует необходимость избавиться от проблемы с предварительным включением (например, сервер закрывается на ключ или находится очень далеко), установите в BIOS параметр питания **Wake After Power Fail** в значение **ON**.

## Другие полезные утилиты для WoL

- **wakeonlan** - утилита для Linux и macOS
- **etherwake** - альтернативная утилита для Linux
- **wolcmd** - утилита для Windows
