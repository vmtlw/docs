---
icon: lucide/network
title: acme.sh
---
# Разбор команд для установки и настройки acme.sh

## 1. Установка необходимых пакетов
``` bash
apt install ssh syslog-ng cron socat logrotate
```
Устанавливаемые пакеты:

  * **ssh** — сервер OpenSSH (для удаленного доступа по SSH).
  * **syslog-ng** — система логирования.
  * **cron** — планировщик задач.
  * **logrotate** — утилита для управления лог-файлами (ротация логов).
  * **socat** — инструмент для перенаправления трафика (полезен при работе с сетевыми сервисами).


## 3. Установка acme.sh для автоматического управления SSL-сертификатами

Клонирование репозитория acme.sh с GitHub.
``` bash
git clone https://github.com/acmesh-official/acme.sh.git && cd ./acme.sh
```

## 4. Установка acme.sh с заданными путями
``` bash 
./acme.sh --install --home /var/lib/acme.sh --config-home /etc/acme.sh --cert-home /etc/ssl/acmesh --accountemail support@mail.ru
```

Пояснение к команде:

  * --home /var/lib/acme.sh — задает каталог установки.
  * --config-home /etc/acme.sh — каталог конфигурации.
  * --cert-home /etc/ssl/acmesh — каталог хранения сертификатов.
  * --accountemail support@mail.ru — указывает email для учетной записи.


## 5. Добавление переменных окружения в .bashrc

Добавим переменные окружения acme.sh в .bashrc, чтобы они загружались при каждом запуске оболочки:
``` bash
cat /var/lib/acme.sh/acme.sh.env >> ~/.bashrc
exec bash
```
## 6. Регистрация аккаунта в Let's Encrypt
``` bash
acme.sh --register-account -m support@mail.ru
```
## 7. Установка Let’s Encrypt в качестве CA (Certification Authority)
``` bash
acme.sh --set-default-ca  --server letsencrypt
```
## 8. Настройка уведомлений в Telegram
``` bash
TELEGRAM_BOT_APITOKEN=11111111111:aaaaaaaaaaaaaaaaaaaaaaaaaa TELEGRAM_BOT_CHATID="-101111111" acme.sh --set-notify --notify-hook telegram --notify-level 1
```

Пояснение к команде:

  * TELEGRAM_BOT_APITOKEN — API-ключ Telegram-бота.
  * TELEGRAM_BOT_CHATID — ID чата, куда будут отправляться уведомления.


## 9. Добавление задания в cron для автоматического обновления сертификатов
``` bash
acme.sh --install-cronjob
```

## 9.1  Генерация сертификата
``` bash
REGRU_API_Username='support@mail.com' REGRU_API_Password='апи_пароль' acme.sh --issue --dns dns_regru -d domain.ru
```

После этой команды логин и пароль от api больше придется указывать при гшенерации , они уже будут зафиксированы в /etc/acme.sh/account.conf


## 10. Установка SSL-сертификата для upload.calculate.ru
``` bash
acme.sh --install-cert -d domain.ru --key-file /etc/nginx/ssl/privkey.pem --fullchain-file /etc/nginx/ssl/fullchain.pem
```
Пояснение

  * -d domain.ru — указывает домен.
  * --key-file /etc/nginx/ssl/privkey.pem — конечный путь для закрытого ключа.
  * --fullchain-file /etc/nginx/ssl/fullchain.pem — конечный путь для полного цепочного сертификата.


### 10.1 А вот такой командой можно задеплоить сертификат на удаленный сервер
``` bash
DEPLOY_SSH_USER="root" DEPLOY_SSH_SERVER="10.0.0.1" DEPLOY_SSH_KEYFILE="/etc/nginx/ssl/privkey.pem" DEPLOY_SSH_FULLCHAIN="/etc/nginx/ssl/fullchain.pem" DEPLOY_SSH_REMOTE_CMD="/etc/init.d/nginx restart" acme.sh --deploy -d *.domain.ru --deploy-hook ssh
```
