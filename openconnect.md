---
title: Настройка openconnect с использованием сертификата для подключения
hide_title: true
---

### Введение
Ocserv (OpenConnect server) - это серверное программное обеспечение для установки VPN-сервера, поддерживающего протокол AnyConnect от Cisco. 

## Установка пакета и настройка основного конфига

1.  Установите сервер ocserv для управления:

```
emerge -av ocserv
```

2. Откройте файл конфигурации `/etc/ocserv/ocserv.conf` и измените следующие параметры:
```
# использование пароля для аутентификации пользователей, ссылаясь на файл /etc/ocserv/ocpasswd для хранения паролей.
enable-auth = "plain[passwd=/etc/ocserv/ocpasswd]"

# аутентификации пользователей должны использоваться сертификаты.
auth = "certificate"

# путь к сертификату центра сертификации (CA), который используется для проверки пользовательских сертификатов
ca-cert = /etc/ocserv/ssl/ca-cert.pem

#  порт, на котором ocserv будет слушать входящие TCP соединения. Порт 443 обычно используется для HTTPS и часто пропускается через файерволы.
tcp-port = 443

# под каким пользователем будет работать ocserv для повышения безопасности.
run-as-user = nobody

# под какой группой будет работать ocserv для повышения безопасности
run-as-group = daemon

# путь к файловому сокету, который используется для межпроцессного взаимодействия.
socket-file = /var/run/ocserv-socket

#  путь к файлу с полным цепочечным сертификатом сервера, выданным Let's Encrypt.
server-cert = /etc/letsencrypt/live/vpn.vmtlw.ru/fullchain.pem

#  путь к приватному ключу сервера, соответствующему сертификату.
server-key = /etc/letsencrypt/live/vpn.vmtlw.ru/privkey.pem

# путь к файлу параметров Диффи-Хеллмана для обеспечения безопасного обмена ключами.
dh-params = /etc/ocserv/dh.pem

# должны ли рабочие процессы быть изолированы друг от друга. Если false, изоляция отключена.
isolate-workers = false

# максимальное количество одновременных подключений от одного и того же пользователя.
max-same-clients = 10

# минимальное время в миллисекундах между запросами для защиты от DoS атак.
rate-limit-ms = 100

#  Время в секундах (7 дней), через которое статистика сервера сбрасывается.
server-stats-reset-time = 604800

#  интервал (в секундах) для отправки keepalive пакетов для поддержания соединения активным.
keepalive = 300

#  интервал (в секундах) для отправки Dead Peer Detection (DPD) пакетов для проверки активности клиента.
dpd = 90

#  интервал (в секундах) для отправки DPD пакетов для мобильных клиентов.
mobile-dpd = 1800

# Время в секундах ожидания перед переключением с UDP на TCP при проблемах с соединением.
switch-to-tcp-timeout = 25

# Определяет, должна ли быть включена автоматическая настройка MTU (Maximum Transmission Unit).
try-mtu-discovery = false

# Определяет OID (Object Identifier) для поиска имени пользователя в сертификате.
cert-user-oid = 0.9.2342.19200300.100.1.1

#  сжатие данных
compression = false

#  Устанавливает приоритеты TLS, запрещая устаревшие версии SSL и TLS (SSL 3.0, TLS 1.0, TLS 1.1)
tls-priorities = "NORMAL:%SERVER_PRECEDENCE:%COMPAT:-VERS-SSL3.0:-VERS-TLS1.0:-VERS-TLS1.1"

# Время (в секундах), через которое неуспешная попытка аутентификации будет завершена.
auth-timeout = 240

# Минимальное время (в секундах) до повторной аутентификации.
min-reauth-time = 300

#  Максимальное количество штрафных баллов перед блокировкой клиента.
max-ban-score = 80

#  Время (в секундах) до сброса бан-листа.
ban-reset-time = 1200

# Время (в секундах), через которое истекает срок действия cookie.
cookie-timeout = 300

# Запрещает роуминг между IP-адресами.
deny-roaming = false

# Время (в секундах) до переподключения с новым ключом (48 часов).
rekey-time = 172800

# Метод переподключения. В данном случае используется SSL.
rekey-method = ssl

# Путь к скрипту, который будет выполняться при подключении клиента
#connect-script = /etc/ocserv/connect-script

# Разрешает использование утилиты occtl для управления сервером.
use-occtl = true

pid-file = /var/run/ocserv.pid
log-level = 2

# Имя виртуального сетевого устройства, используемого для VPN.
device = vpns

#  Определяет, будут ли IP-адреса предсказуемыми для клиентов
predictable-ips = true

# Устанавливает домен
default-domain = "vpn.vmtlw.ru"

# Устанавливает подсеть IPv4 для VPN-клиентов.
ipv4-network = 10.8.1.0/24

# Указывает DNS-сервер, который будет использоваться VPN-клиентами
dns = 1.1.1.1

#  Указывает, должен ли сервер пинговать IP-адреса перед выдачей их клиентам.
ping-leases = false

#  Устанавливает маршрут по умолчанию для всех клиентских подключений.
route = default

#  Путь к директории, содержащей индивидуальные конфигурации для каждого пользователя.
config-per-user = /etc/ocserv/config-per-user/

# Включает совместимость с Cisco AnyConnect клиентами.
cisco-client-compat = true

# Включает поддержку устаревшего DTLS (Datagram Transport Layer Security).
dtls-legacy = true

# Отключает совместимость с Cisco SVC клиентами.
cisco-svc-client-compat = false

#  Запрещает клиентам обходить VPN протокол
client-bypass-protocol = false

```

3. Добавьте пользователей:
```
ocpasswd -c /etc/ocserv/ocpasswd yourusername
```
4. Настройте firewall:
Разрешите порты 443 (или другой, который вы используете) для входящих подключений, также чтобы направить трафик клиента в Интернет через NAT, выполните:
```
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
iptables -t nat -A POSTROUTING -s {vpn подсеть}/{маска} -o {имя интерфейса, через который есть выход в интернет} -j MASQUERADE
/etc/init.d/iptables save
rc-update add iptables

```

5. разрешите пересылку трафика между интерфейсами системы а так же ускорьте свой vpn:
Для этого откройке файл `/etc/sysctl.conf` и внесите в него следующие строчки:
```
# Включает пересылку IPv4-пакетов в системе, позволяя машине пересылать пакеты, не предназначенные для неё, таким образом, она действует как маршрутизатор.
net.ipv4.ip_forward=1

# Аналогично первой строке, включает пересылку для всех IPv4 сетевых интерфейсов. Это обеспечивает глобальное включение пересылки пакетов на всех интерфейсах.
net.ipv4.conf.all.forwarding=1

# Включает пересылку IPv6-пакетов в системе, позволяя машине пересылать пакеты IPv6, делая её маршрутизатором для IPv6.
net.ipv6.conf.all.forwarding=1

# Устанавливает дисциплину очередей по умолчанию (qdisc) для сетевых интерфейсов на Fair Queuing (fq). Это помогает достичь справедливого распределения пропускной способности и управления перегрузками.
net.core.default_qdisc=fq

# алгоритм управления перегрузками TCP на BBR (Bottleneck Bandwidth and Round-trip propagation time). BBR может значительно улучшить пропускную способность и уменьшить задержки в сетевых коммуникациях.
net.ipv4.tcp_congestion_control=bbr
```
Примените изменения немедленно, без необходимости перезагрузки, выполнив команду:
```
sysctl -p
```
6. Добавьте сервер openconnect в автозагрузку и запустите:
```
rc-update add openrc
/etc/init.d/ocserv start
```

## Настройка авторизации по сертификату на сервере OpenConnect VPN (ocserv)
В этом руководстве будет показано, как настроить аутентификацию по сертификату на VPN-сервере OpenConnect (ocserv) Linux. 
OpenConnect (ocserv) — это реализация протокола Cisco AnyConnect VPN с открытым исходным кодом.

Каждый раз вводить имя пользователя и пароль может быть затруднительно, особенно если клиентское программное обеспечение, такое как приложение Cisco AnyConnect для iOS, не предлагает возможности запоминать пароли. Многие клиентские программы OpenConnect могут импортировать пользовательские сертификаты, что освобождает пользователя от необходимости вводить имя пользователя и пароль. Аутентификация по сертификату также более безопасна, чем аутентификация по паролю.

### Требования
При работе с этим руководством предполагается, что вы уже настроили VPN-сервер OpenConnect с сертификатом сервера Let's Encrypt TLS.

Мы создадим собственный CA (центр сертификации) для подписи сертификата клиента. Демон ocservдолжен продолжать использовать сертификат сервера TLS, выданный Let's Encrypt, чтобы клиентское программное обеспечение не отображало предупреждение системы безопасности.

Мы хотим использовать аутентификацию по сертификату, но Let's Encrypt не выдает клиентский сертификат, поэтому нам нужно создать собственный центр сертификации. Вы можете использовать openssl для этой работы, но ocserv рекомендует GnuTLS, поэтому я покажу вам, как использовать GnuTLS.

### Настройка собственного CA (центра сертификации)
Установите gnutls:
```
emerge -v net-libs/gnutls
```

Создайте подкаталог /etc/ocserv/ssl
```
mkdir -p /etc/ocserv/ssl && cd /etc/ocserv/ssl`
```

Создайте закрытый ключ для центра сертификации с помощью certtool. По умолчанию он генерирует 3072-битный ключ RSA, чего вполне достаточно.
```
certtool --generate-privkey \
--outfile ca-privkey.pem
```

Прежде чем создавать сертификат CA, давайте создадим файл шаблона сертификата CA. Формат файла шаблона можно найти в руководстве по certtool ( man certtool).
```
vi ca-cert.cfg
```
Добавьте в файл следующие строки. Замените заполнители соответствующими значениями.

```
# X.509 Certificate options

# Организация. 
organization = "domain.example.com"

# Общее имя владельца сертификата. 
cn = "example.com CA"

# Серийный номер сертификата. 
serial = 001

# Через сколько дней, считая с сегодняшнего дня, истечет срок действия этого сертификата. Используйте -1, если срок действия не указан
expiration_days = -1

# Является ли это сертификатом CA или нет 
ca

# Будет ли этот сертификат использоваться для подписи данных 
signing_key

# Будет ли этот ключ использоваться для подписи других сертификатов. 
cert_signing_key

# Будет ли этот ключ использоваться для подписания CRL. 
crl_signing_key
```
Сохраните и закройте файл. Теперь сгенерируйте сертификат CA, используя конфигурации из файла шаблона.
```
certtool --generate-self-signed \
--load-privkey ca-privkey.pem \
--template ca-cert.cfg \
--outfile ca-cert.pem
```
Теперь у нас есть файл сертификата CA ( ca-cert.pem).


### Генерация сертификата клиента

Теперь выполните следующую команду, чтобы сгенерировать закрытый ключ клиента.
```
certtool --generate-privkey \
--outfile client-privkey.pem
```

Создайте файл шаблона сертификата клиента.
```
vim client-cert.cfg
```

Добавьте в файл следующие строки. Uid должен быть именем пользователя в файле `/etc/ocserv/ocpasswd`.
```
# X.509 Certificate options

organization = "vpn.example.com" 

# Общее имя владельца сертификата. 
cn = "Ivan Petrov" 

# Идентификатор пользователя владельца сертификата. 
uid = "username" 

# Через сколько дней, считая с сегодняшнего дня, истечет срок действия этого сертификата. Используйте -1, если срок действия не указан. 
expiration_days = 3650 

# Будет ли этот сертификат использоваться для TLS-сервера 
tls_www_client 

# Будет ли этот сертификат использоваться для подписи 
signing_key

# Будет ли этот сертификат использоваться для шифрования данных (необходим в наборах шифров TLS RSA). Обратите внимание, что для шифрования и подписи предпочтительно использовать разные ключи. 
encryption_key
```
Сохраните и закройте файл. Затем выполните следующую команду, чтобы сгенерировать сертификат клиента, который будет подписан закрытым ключом CA.
```
certtool --generate-certificate \
--load-privkey client-privkey.pem \
--load-ca-certificate ca-cert.pem \
--load-ca-privkey ca-privkey.pem \
--template client-cert.cfg \
--outfile client-cert.pem
```

Объедините закрытый ключ клиента и сертификат в файл PKCS #12, защищенный PIN-кодом.
```
certtool --to-p12 \
--load-privkey client-privkey.pem \
--load-certificate client-cert.pem \
--pkcs-cipher aes-256 \
--outfile client.p12 \
--outder \
--p12-name admin \
--empty-password
```
                                        
### aутентификация сертификата ocserv

Теперь у нас есть закрытый ключ клиента и сертификат, объединенные в один файл client.p12.

Обратите внимание, что приложение Ciso AnyConnect для iOS не поддерживает шифр AES-256. Он откажется импортировать сертификат клиента. Если пользователь использует устройство iOS, вы можете выбрать шифр 3des-pkcs12.
```
certtool --to-p12 \
--load-privkey client-privkey.pem \
--load-certificate client-cert.pem \
--pkcs-cipher 3des-pkcs12 \
--outfile ios-client.p12 \
--outder \
--p12-name admin \
--hash SHA1
```
Закрытый ключ клиента и сертификат объединены в один файл ios-client.p12.

                            
### Запрос на подпись сертификата

Этот шаг необходим только в том случае, если имеется несколько пользователей VPN и пользователь хочет использовать свой собственный закрытый ключ.

Чтобы сохранить секретные ключи конечных пользователей, пользователи могут генерировать запрос на подпись сертификата (CSR) со своими собственными секретными ключами, а затем отправлять запросы на сертификаты администратору, который затем выдает пользователям клиентские сертификаты. Сначала им необходимо сгенерировать закрытый ключ и шаблон сертификата клиента, используя команды, упомянутые выше. Затем сгенерируйте CSR с помощью следующей команды. Файл request.pemподписан закрытым ключом пользователя.
```
certtool --generate-request \
--load-privkey client-privkey.pem \
--template client-cert.cfg \
--outfile request.pem
```

Затем пользователь отправляет request.pem файл client-cert.cfg администратору, который запускает следующую команду для создания сертификата клиента.
```
certtool --generate-certificate \
--load-ca-certificate ca-cert.pem \
--load-ca-privkey ca-privkey.pem\
--load-request request.pem \
--template client-cert.cfg \
--outfile client-cert.pem
```

После этого администратор отправляет client-cert.pem пользователю файл сертификата.

### Включение аутентификации сертификатов в демоне ocserv
Отредактируйте файл конфигурации ocserv.
```
vim /etc/ocserv/ocserv.conf
```

Чтобы включить аутентификацию сертификата, раскомментируйте следующую строку.
```
auth = "plain[passwd=/etc/ocserv/ocpasswd]"
auth = "certificate"
```

Если обе приведенные выше строки не закомментированы, это означает, что пользователь должен пройти аутентификацию по паролю и аутентификацию по сертификату. Поэтому, если аутентификации по сертификату достаточно для подтверждения личности, закомментируйте первую строку.

Если вы разрешаете пользователям выбирать аутентификацию по сертификату или аутентификацию по паролю, вместо этого у вас должны быть следующие строки.
```
enable-auth = "plain[passwd=/etc/ocserv/ocpasswd]" 
auth = "certificate"
```

Нам нужно использовать наш собственный сертификат CA для проверки сертификата клиента, поэтому измените эту строку
```
ca-cert = /etc/ocserv/ssl/ca-cert.pem
```

Далее найдите следующую строку.
```
cert-user-oid = 0.9.2342.19200300.100.1.1
```
Вам не нужно его менять. `0.9.2342.19200300.100.1.1` это UID, указанный в сертификате клиента. Приведенная выше строка сообщает ocserv демону, что ему нужно найти имя пользователя в поле UID сертификата клиента. Если сертификат клиента успешно проверен сертификатом CA и ocservдемон может найти соответствующее имя пользователя в файле `/etc/ocserv/ocpasswd`, клиент сможет войти в систему.

Сохраните и закройте файл. Затем перезапустите ocserv.
```
/etc/init.d/ocserv restart
```

### Использование аутентификации сертификата на Linux
Загрузите client.p12файл на Linux
```
scp user@vpn.example.com :/etc/ocserv/ssl/client.p12 ~
```

Затем установите openconnect
```
emerge openconnect
```

Чтобы использовать аутентификацию сертификата, запустите
```
openconnect -b vpn.example.com -c client.p12
```
Вам будет предложено разблокировать закрытый ключ клиента с помощью PIN кода, который вы установили ранее в этом руководстве. Если PIN код введен правильно, вы должны быть подключены к VPN-серверу.


### Использование аутентификации сертификата на устройстве iOS
Пользователи iOS могут использовать приложение Cisco AnyConnect. Чтобы импортировать сертификат клиента в приложение AnyConnect, вы можете сначала отправить файл PKCS # 12 на свой адрес электронной почты во вложении. Затем откройте почтовое приложение на iOS. Нажмите на вложение на несколько секунд и поделитесь им с AnyConnect. Затем введите PIN-код для импорта файла. После импорта отредактируйте свое VPN-соединение в AnyConnect. Перейдите в Advanced-> Certificateи выберите сертификат клиента. Сохраните настройки. Теперь вам больше не нужно вводить имя пользователя и пароль на вашем устройстве iOS. Приложение Cisco AnyConnect не запоминает имя пользователя и пароль, поэтому в режиме аутентификации по паролю VPN-соединение прерывается, когда телефон не используется. В режиме аутентификации по сертификату приложение автоматически повторно подключится к VPN-серверу, если соединение будет разорвано.

### Проблемы с клиентом AnyConnect на iOS
В последней версии клиента AnyConnect для iOS возникла проблема при использовании аутентификации сертификата в протоколе TLS 1.3. 
Если вы видите ошибку в журнале ocserv `GnuTLS error (at worker-vpn.c:795): A TLS fatal alert has been received.`, у вас та же проблема. Вам нужно либо использовать аутентификацию по паролю в клиенте AnyConnect iOS, либо отключить TLS 1.3 в файле конфигурации ocserv. Чтобы отключить TLS1.3, найдите tls-priorities параметр в /etc/ocserv/ocserv.confфайле и добавьте :-VERS-TLS1.3 в конец, чтобы отключить TLS 1.3.
```
tls-priorities = tls-priorities = "NORMAL:%SERVER_PRECEDENCE:%COMPAT:-RSA:-VERS-SSL3.0:-ARCFOUR-128:-VERS-TLS1.0:-VERS-TLS1.1:-VERS-TLS1.3"
```
Сохраните и закройте файл. Затем перезапустите ocserv.
```
/etc/init.d/ocserv restart
```



### Примечание
Если вы видите фразу с `SSL 3.3` в журналах ocserv, не паникуйте. SSL 3.3 — это не TLS 1.2. Вы используете безопасное соединение TLS.
