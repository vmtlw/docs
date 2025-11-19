---
title: LightDM FIDO2
icon: lucide/hard-drive
---

### Введение

Данная инструкция позволит вам входить по физическому электронному ключу , например Yubikey


Установить нужные пакеты:
``` bash
apt install pamtester libpam-u2f pamu2fcfg
```

Создаем каталог для ключей и сгенерируем ключ
``` bash
mkdir -p ~/.config/Yubico
chmod 700 ~/.config/Yubico
```

Подключаем FIDO2-ключ и генерируем файл ключей:
``` bash
pamu2fcfg > ~/.config/Yubico/u2f_keys
```

!!! note
    реагируем на состояение ключа


Проверьте содержимое:
``` bash
cat ~/.config/Yubico/u2f_keys
```

### Включаем поддержку в PAM (LightDM)
Самое важное — включить FIDO2 до обычного пароля, но не делать систему непригодной, если ключа нет.

Открываем PAM-конфиг LightDM /etc/pam.d/common-auth :
Скорее всего там будет что-то вроде:
``` bash
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
``` bash
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

Для того чтобы при подключении по ssh была дополнительная проверка по физическому ключу необходимо создать пару ключей ed25519_sk и при создании привязать их к нашему физическому ключу:

```
ssh-keygen -t ed25519-sk -O resident -O verify-required
Generating public/private ed25519-sk key pair.
You may need to touch your authenticator to authorize key generation.
Enter PIN for authenticator: 
You may need to touch your authenticator again to authorize key generation.
Enter file in which to save the key (/home/rooty/.ssh/id_ed25519_sk): 
Enter passphrase for "/home/rooty/.ssh/id_ed25519_sk" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/rooty/.ssh/id_ed25519_sk
Your public key has been saved in /home/rooty/.ssh/id_ed25519_sk.pub
The key fingerprint is:
SHA256:R5EJYMyOLf3Ms47nhw0IgFQVpRfA46dconur8emG2jA rooty@desktop
The key's randomart image is:
+[ED25519-SK 256]-+
```

А далее можно раскидать открытый ключ id_ed25519_sk.pub по нашим серверам

При попытке подключения по ssh на удаленном сервере c с использованием созданного нами d_ed25519_sk будет использован фищзический ключ 
