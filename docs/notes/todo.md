---
icon: lucide/list-todo
title: Теоретический потенциал физического электронного ключа

---

### 1. GPG (OpenPGP) на YubiKey — must-have для любого devops
Самый мощный кейс — хранить GPG subkeys на YubiKey.
Сценарии:

✔️ Подпись пакетов (DEB/RPM), пакетов репозиториев дистрибутива
YubiKey как HSM:
хранит приватный GPG-ключ внутри (невозможно экспортировать);
идеально подходит для подписи пакетов при сборках.
Если ты участвуешь в разработке дистрибутива — это реально промышленный стандарт.

✔️ Git commit signing (podpis коммитов GPG)
С подписями коммитов + GitHub/GitLab Verified Badge.

✔️ Подпись артефактов в CI/CD
Можно выдавать runner-ам временный доступ (через touch-to-sign), чтобы:
подписывать релизы,
подписывать контейнеры (через cosign можно использовать GPG).

### 2. SSH через FIDO2 (ssh -o SecurityKeyProvider)
Если у тебя уже есть один SSH-ключ на YubiKey — можно усилить сценарии:

✔️ FIDO2 resident keys (ключи, хранящиеся внутри yubikey)
Позволяют подключаться без ~/.ssh/id_* на диске.
Можно пойти дальше:

✔️ SSH с PIN + Touch
То есть:
PIN → защита от подбора боковым софтом
Touch → защита от malware
Сам приватный ключ → на YubiKey
По уровню защиты — почти как SmartCard.

### 3. PIV / SmartCard (PKCS#11) — корпоративный уровень
Можно использовать YubiKey как:

✔️ PKCS#11 токен для OpenSSH (через ssh -I /usr/lib/.../pkcs11.so)
Эта схема отличается от FIDO2: приватный ключ создаётся как RSA/ECDSA на самом токене.
✔️ Аутентификация в VPN:
OpenVPN (через PKCS#11)
strongSwan IPsec (EAP-TLS)
WireGuard через агенты (непрямо, но можно)

✔️ S/MIME подпись почты (Thunderbird, mutt)
### 4. PAM + FIDO2 → вход в Linux через ключ
Это ты уже упоминал, но можно улучшить:

✔️ Полное отключение пароля (passwordless)
Используя:

bash
￼Копировать код
fido2-token -L /dev/hidrawX
pamu2fcfg > ~/.config/fido/pam-u2f
✔️ Multi-factor login:
пароль + YubiKey touch
или

PIN + YubiKey touch

✔️ Sudo с FIDO2 (да, можно)
Используя pam_u2f для sudo.

### 5. LUKS + YubiKey: аппаратная расшифровка дисков
Есть несколько режимов:

✔️ challenge-response для LUKS
Схема:

YubiKey генерирует HMAC-SHA1 или FIDO2 challenge-response
Результат → ключ для LUKS
Это очень мощно, т.к. без физического ключа диск не расшифруется вообще.
✔️ LUKS + YubiKey + TPM (комбинированный unlock)
Можно делать advanced схемы для дистрибутива/серверов.

### 6. 2FA для sudo без пароля
Можно сделать:
sudo → asks YubiKey touch only
Сценарий удобен для продакшен-серверов:
ты залогинен по SSH ключу,
но sudo требует физическую авторизацию.

### 7. Подпись контейнеров (cosign/sigstore) через YubiKey
cosign может использовать:
PIV (pkcs11)
GPG (через subkey)
FIDO2 (относительно новые фичи)
Можно подписывать образы:

nginx
￼Копировать код
cosign sign --key yubikey:// ...
И проверять их при деплое в Kubernetes.

### 8. Secure bootstrap / Secure update system для дистрибутива
Если ты разрабатываешь свой Linux-дистрибутив:

✔️ YubiKey как root-of-trust
Хранишь:
ключ подписи репозиториев,
ключ подписи образов ISO,
ключ подписи обновлений.
Touch-to-sign делает это безопаснее обычного HSM.

### 9. Автогенерация OTP для Linux-доступов
Используется реже, но можно:
✔️ Yubikey → OTP → login via pam_yubico
Работает через YubiCloud или локальный YubiHSM.```
