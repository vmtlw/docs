---
title: IPsec
icon: lucide/globe-lock
---
# VPN настройка с использованием StrongSwan

Этот документ описывает настройку VPN сервера с использованием StrongSwan и IPsec, с использованием предустановленных ключей (PSK) для аутентификации.

## Установка StrongSwan

Для установки StrongSwan, выполните следующую команду:

``` bash
apt install strongswan strongswan-pki strongswan-libcharon strongswan-charon libstrongswan-extra-plugins libcharon-extauth-plugins
```

#### Конфигурация IPsec /etc/ipsec.conf
Настройте файл /etc/ipsec.conf следующим образом:

``` title="/etc/ipsec.conf"
config setup
    charondebug="ike 2, knl 2, cfg 2, net 2"

conn ikev2-psk
    keyexchange=ikev2
    type=tunnel
    auto=add
    dpdaction=clear
    dpddelay=30s
    rekey=no
    fragmentation=yes
    forceencaps=yes

    # Локальная сторона
    left=%any
    leftid=89.179.122.11
    leftauth=psk
    leftsubnet=0.0.0.0/0

    # Удалённая сторона (клиент)
    right=%any
    rightid=%any
    rightauth=psk
    rightsourceip=10.10.10.0/24
    ike=chacha20poly1305-sha512-curve25519-prfsha512,aes256gcm16-sha384-prfsha384-ecp384,aes256-sha1-modp1024,aes128-sha1-modp1024,3des-sha1-modp1024!
    esp=chacha20poly1305-sha512,aes256gcm16-ecp384,aes256-sha256,aes256-sha1,3des-sha1!

```

Для конфигурации ключа PSK, отредактируйте файл /etc/ipsec.secrets и добавьте строку:

``` title="/etc/ipsec.secrets"
: PSK "password-strong-line"
```

#### Структура каталогов для сертификатов
В каталоге /etc/ipsec.d/ должны быть следующие подкаталоги для сертификатов и ключей:

``` bash title="tree /etc/ipsec.d/"
/etc/ipsec.d/
├── aacerts
├── acerts
├── cacerts
├── certs
├── crls
├── ocspcerts
├── policies
├── private
└── reqs
```

#### Настройка iptables

Чтобы очистить существующие правила, выполните следующие команды:

``` bash
iptables -L
iptables -F
iptables -X
iptables -Z
iptables -t nat -F
iptables -t nat -X
iptables -t nat -Z
iptables -t mangle -F
iptables -t mangle -X
iptables -t mangle -Z
```

Настройте iptables для разрешения трафика VPN:

``` bash
iptables -A FORWARD -s 10.10.10.0/24 -o br0 -j ACCEPT
iptables -A FORWARD -d 10.10.10.0/24 -i br0 -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o br0 -j MASQUERADE
```
Так же если сервер находится за NAT то нужно пробросить порты:
```
UDP/4500 - NAT-T
UDP-500 - IKE
IPSEC-ESP (без указанрия порта)
```

Теперь ваша VPN-сеть должна быть настроена и готова к работе.

