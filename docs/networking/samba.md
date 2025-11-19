---
title: Samba
icon: lucide/globe-lock
---
# Конфиг шаринга самбы (1 публичной и 1 приватной папки , с поддежкой unix simlinks )

```bash
[global]
allow insecure wide links = yes
use sendfile = yes
deadtime = 30
server multi channel support = yes

[downloads]
path = /mnt/sda1/downloads
public = yes
writable = yes
force create mode = 0666
force directory mode = 0777
browseable = yes
follow symlinks = yes
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
