
---
title: LightDM FIDO2
icon: lucide/hard-drive
---

## Введение
Данная инструкция позволит вам входить по физическому электронному ключу , например Yubikey


1. Установить нужные пакеты
```
apt install pamtester libpam-u2f pamu2fcfg
```

Создаем каталог для ключей и сгенерируем ключ
```
mkdir -p ~/.config/Yubico
chmod 700 ~/.config/Yubico
```

Подключаем FIDO2-ключ и генерируем файл ключей:
```
pamu2fcfg > ~/.config/Yubico/u2f_keys
```

!!! note
    реагируем на состояение ключа


Проверьте содержимое:

```
cat ~/.config/Yubico/u2f_keys
```


### Включаем поддержку в PAM (LightDM)
Самое важное — включить FIDO2 до обычного пароля, но не делать систему непригодной, если ключа нет.

Открываем PAM-конфиг LightDM /etc/pam.d/common-auth :
Скорее всего там будет что-то вроде:
```
auth    [success=1 default=ignore]      pam_unix.so nullok
auth    requisite                       pam_deny.so
auth    required                        pam_permit.so
auth    optional
                       pam_cap.so
```

Добавляем строку для FIDO2
Если хочешь необязательный FIDO2 (fallback на пароль):

И помещаем перед pam_unix.so:

Пример:
```
auth    sufficient    pam_u2f.so cue
auth    [success=1 default=ignore]      pam_unix.so nullok
auth    requisite                       pam_deny.so
auth    required                        pam_permit.so
auth    optional                        pam_cap.so
```


 Если хочешь сделать FIDO2 обязательным:
```
auth    required    pam_u2f.so cue
```
Но тогда без ключа входа нет вообще.

Тест до выхода из сессии
Проверь вход PAM:
```
sudo pamtester login $USER authenticate
sudo pamtester lightdm $USER authenticate
```


