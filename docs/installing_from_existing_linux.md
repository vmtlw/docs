---
title: install RAM linux
icon: lucide/terminal
---

Здесь вы узнаете, как можно установить Calculate Linux из уже
работающего хоста Linux.

Идея проста. Скачайте свежую версию Calculate Linux в корень файловой
системы:

```bash title="wget --quiet --show-progress  https://mirror.yandex.ru/calculate/nightly/20240330/css-20240330-x86_64.iso -P /tmp"

css-20240330-x86_64.iso        0%[                                               ]  11,97M  18,42MB/s
```

Вместо 20240330 подставьте актуальную версию образа. Посмотреть его
можно [по ссылке](https://mirror.yandex.ru/calculate/release/).

Если у вас удалённый сервер, получите KVM-консоль, после чего
перезагрузите систему. См. рисунок ниже:


В загрузчике нажмите клавишу `"c"` для перехода в консоль Grub и впишите
следующий текст:

```
    loopback loop /css-20240330-x86_64.iso
    linux (loop)/boot/vmlinuz root=live iso-scan/filename=/css-20240330-x86_64.iso rd.plymouth=0 rd.live.ram
    initrd (loop)/boot/initrd
    boot
```


После этого будет загружен в память и запущен образ Calculate Linux. Для
запуска и установки приведённого в примере Calculate Scratch Server
достаточно 1 Гб оперативной памяти.

Теперь можно приступить к установке Calculate Linux на диск:

```bash 
cl-install -D /dev/vda -S data -l ru_RU --timezone Europe/Moscow --hostname my.domain.org
```

В примере на диске /dev/vda создаётся корневой раздел и раздел с
данными, настраивается локализация ru_RU, часовой пояс
Europe/Moscow и сетевое имя машины my.domain.org.

