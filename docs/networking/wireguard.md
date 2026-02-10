1. Установка пакетов
Перейдите в LuCI → System → Software и установите пакет luci-proto-wireguard.

При желании установите пакет qrencode, чтобы иметь возможность создания QR-кода при настройке пира для простого импорта на мобильный клиент wireguard.

2. Перезапуск служб
Перейдите в LuCI → System → Startup → Initscripts и нажмите на network → Restart.

3. Добавление сетевого интерфейса WireGuard
Чтобы создать новый интерфейс WireGuard, перейдите в LuCI → Network → Interfaces → Add new interface...

Выберите WireGuard VPN из выпадающего меню Protocol.
Назовите интерфейс wg0 (или как предпочитаете)
Нажмите на Create Interface, чтобы создать его и открыть для редактирования
4. Настройка сетевого интерфейса WireGuard
В открывшемся окне редактирования интерфейса настройте следующее:

Нажмите на Generate new key pair, чтобы сгенерировать приватный и публичный ключи
Listen port: 51820 или предпочитаемый порт
IP addresses: 10.0.0.1/24 или предпочитаемый внутренний VPN IPv4-адрес для интерфейса WireGuard-сервера
Сохраните эту конфигурацию
5. Настройка пиров WireGuard
Чтобы создать новую конфигурацию пира WireGuard, перейдите в LuCI → Network → Interfaces → wg0 → Edit → Peers

Нажмите на Add peer
Нажмите на Generate new key pair, чтобы заполнить поля публичного и приватного ключей
Allowed IPs: 10.0.0.10/32 или любой другой адрес, который вы назначите клиенту
Endpoint port: 51820
Persistent Keep Alive: 25
Сохраните
Нажмите на Edit для только что созданного пира

Нажмите на Generate configuration... и в разделе Connection endpoint выберите:
* Если подключение с общедоступного IPv4-адреса - IPv4-адрес wan-интерфейса роутера
* Если подключение с общедоступного IPv6-адреса - IPv6-адрес wan-интерфейса роутера
* Если используется общедоступное доменное имя, введите его как пользовательскую запись
Для передачи конфигурации пира на клиентское устройство:

Используйте клиент Wireguard на телефоне/планшете, который может сканировать сгенерированный QR-код, или
Скопируйте и вставьте сгенерированные данные конфигурации в файл device.conf для импорта в клиент WireGuard
После сохранения изменений в интерфейсе wg0, нажмите Save & Apply на странице Interfaces, затем Restart wg0. Это необходимо для применения нового списка пиров. Только "Save & Apply" недостаточно!

6. Настройка брандмауэра для трафика WireGuard
Перейдите в LuCI → Network → Firewall → General Settings и в разделе Zones добавьте новую зону:

Name: WireguardVPN (или предпочитаемое имя)
Input: accept
Output: accept
Intra zone forward: accept
Masquerading: отмечено
MSS Clamping: отмечено
Covered networks: wg0
Allow forward to destination zones: lan и wan
Allow forward from source zones: lan
Сохраните
Создайте правило для разрешения прохождения трафика IPv4 и IPv6 из интернета для подключения с клиентского устройства, использующего IPv4 (если у роутера есть публичный IPv4-адрес) или с клиентского устройства, использующего IPv6 (если у роутера есть доступный публичный IPv6-адрес).

Перейдите в LuCI → Network → Firewall → Traffic Rules

Name: WireGuard-incoming (или предпочитаемое имя)
Protocol: UDP
Source zone: wan
Source address: -- add IP --
Source port: any
Destination zone: Device
Destination address: -- add IP--
Destination port: 51820
Action: accept
Сохраните, Save & apply
Примечание: Если для подключения к серверу WireGuard используется только IPv4, вышеуказанное правило трафика брандмауэра можно заменить правилом переадресации портов.

Если между роутером Openwrt, настроенным как сервер WireGuard, и интернетом есть вышестоящий ISP-роутер, то порт 51820 также должен быть открыт для трафика IPv4/IPv6 к роутеру Openwrt.

Тестирование
Установите VPN-соединение. Проверьте маршрутизацию с помощью traceroute и traceroute6.

traceroute openwrt.org
traceroute6 openwrt.org
Проверьте свой IP и DNS-провайдера.

ipleak.net
dnsleaktest.com
На роутере:

Перейдите в LuCI > Status > Wireguard и проверьте, подключено ли устройство пира с IPv4 или IPv6-адресом и с недавним временем рукопожатия
Перейдите в LuCI > Network > Diagnostics и выполните ipv4 ping на IP-адрес клиентского устройства, например 10.0.0.10
На клиентском устройстве в зависимости от программного обеспечения wireguard:

Проверьте трафик передачи для tx и rx
Выполните ping на внутренний LAN IP-адрес роутера
Проверьте публичный IP-адрес в браузере – https://whatsmyip.com – должен отображаться публичный IP-адрес ISP для роутера
Устранение неисправностей
Соберите и проанализируйте следующую информацию.

# Перезапустите службы
service log restart; service network restart; sleep 10
 
# Журнал и статус
logread -e vpn; netstat -l -n -p | grep -e "^udp\s.*\s-$"
 
# Конфигурация во время выполнения
pgrep -f -a wg; wg show; wg showconf wg0
ip address show; ip route show table all
ip rule show; ip -6 rule show; nft list ruleset
