---
title: certbot
icon: lucide/cpu
---

#### получаем wildcard 1 командой

``` bash
certbot certonly \
    --manual \
    --preferred-challenges dns \
    --server https://acme-v02.api.letsencrypt.org/directory \
    -d "*.domain.com"
```

#### Добавляем отображенную txt запись и далее ждем когда она расползется по dns
Чекать можно по:
```
dig txt _acme-challenge.domain.com 8.8.8.8 +short
```
если получаем что то вроде tEwx_TOI_ckwqLHMByG-nKITITeVGfIkRKk7qMYhbnE то на всякий случяай проверь еще 1.1.1.1 если и там все ок то продолжаем
!!! note
    после получаения не забудь удалить dns txt запись 

