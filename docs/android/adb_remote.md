---
title: Android Debug Bridge 
icon: lucide/smartphone
---

## Требования
Для подключения к adb-устройству через сеть необходимы следующие опции:
1.  запущенный adb-демон
2.  разрешение для подключения по сети
3.  Клиенту с его adb-ключом должен быть предоставлен доступ к
    устройству

## Управление ключами для подключения
Возможность безнаказанного подключения к ПК управляется опцией `ro.adb.secure`
Для получения возможности без ограничений подключаться к adb подключитесь к устройству и выполните следующую последовательность команд:

```
adb root
adb remount /vendor
adb disable-verity
adb reboot
```

Чтобы заблокировать доступ неавторизированным пользователям, необходимо выполнить обратную команду:

```
adb enable-verity
```


### Альтернативный способ управления проверкой ключей
В некоторых случаях возможно, что команда `disable-verity` не изменит ситуацию. Такое наблюдается, например, в коробочках `X96 Max+`. В таком случае необходимо изменять параметры в файлах `build.prop` вручную.
Для включения проверки выполните команду 

```
grep ro.adb.secure /vendor/build.prop && sed -i'/ro.adb.secure/s/0/1/' /vendor/build.prop || echo ro.adb.secure=1 >> /vendor/build.prop
```

Для выключения проверки выполните команду

```bash 
grep ro.adb.secure /vendor/build.prop && sed -i '/ro.adb.secure/s/1/0/' /vendor/build.prop || echo ro.adb.secure=0 >> /vendor/build.prop
```

### Добавление ключей доступа
После запуска adbd на клиентской машине, в папке `$HOME` появляется пара приватного и публичного ключа. Список авторизованных ключей хранится на устройстве в директории [/data/misc/adb/adb_keys]
Добавьте клиентский ключ доступа, взяв его из `$HOME/.android/adbkey.pub`

```bash
adb root
cat ~/.android/adbkey.pub | adb shell tee -a /data/misc/adb/adb_keys
adb reboot
```

Для исключения ключа root@pchost из списка доступа аппарата возможно с помощью команды

```bash
adb root
adb shell sed -i '/root@pchost/d' /data/misc/adb/adb_keys
adb reboot
```

## Изменение порта доступа
Измените порт подключения adb на подключенном устройстве на 5555:

```bash
adb root
adb remount /vendor
adb shell
grep service.adb.tcp.port /vendor/build.prop && sed -i '/service.adb.tcp.port/s/=.*/=5555/' /vendor/build.prop || echo service.adb.tcp.port=5555 >> /vendor/build.prop
reboot
```

### Подключение к устройству с нестандартным портом
Подключитесь к удаленному adb-интерфейсу устройства с адресом 10.2.19.50 и с открытым портом 5555 с помощью команды`adb connect 10.2.19.50:5555`

`connected to 10.2.19.50:5555`

### AOSP на Mi A2 Lite
Запустите возможность отладки в меню разработчиков
Подключите аппарат к компьютеру и разрешите модификацию системных
файлов:

```
adb root
adb remount /system
adb remount /vendor
```

Подключитесь к устройству и включите соединение по сети и проверку adb-клиента по ключу:

```bash title="adb shell"
grep service.adb.tcp.port /vendor/build.prop && sed -i '/service.adb.tcp.port/s/=.*/=5555/' /vendor/build.prop || echo service.adb.tcp.port=5555 >> /vendor/build.prop
sed -i 's/adb.secure=0/adb.secure=1/'
/system/etc/prop.default
reboot
```
Отключите аппарат от компьютера, дождитесь перезагрузки и подключитесь к
устройству по адресу 10.2.19.2:5555
adb connect 10.2.19.2:5555
Проверьте подключение

```
adb shell
phhgsi_arm64_ab:/ $ exit
```
### Samsung Galaxy Tab A7 (2020)
Для запуска adb по сети в устройстве необходимо вносить изменения в
раздел `/system`, а не `/vendor`

#### Разблокируйте загрузчик
Откроёте меню разработчиков, выберите там пункт **заводская
разблокировка**.
Перезагрузите планшет в режим **Download mode**, для этого перезагрузите
аппарат, удерживая одновременно обе клавиши громкости.
Согласитесь на удаление данные и разблокировку загрузчика, долгим
нажатием клавиши увеличения громкости.

#### Получите root
Скачайте оригинальную прошивку, [Magisk Manager](https://github.com/topjohnwu/Magisk/releases/latest), [драйверы](https://developer.samsung.com/mobile/android-usb-driver.html) для windows, а также утилиту [Odin](https://4pda.ru/forum/index.php?showtopic=648344)
Скачайте прошивку с помощью утилиты [samloader](https://github.com/nlscc/samloader) или попробуйте ее найти на [сайте](https://samfw.com/firmware/SM-A325F/SER)
Подготовьте место для загрузки

```
mkdir samloader
cd samloader
python3 -m venv env
source env/bin/activate
pip3 install --upgrade pip
pip3 install git+https://github.com/nlscc/samloader.git
```

Получите название последней версии
```
samloader -m SM-T505 -r SER checkupdate `T505XXS2ATL1/T505OXM2ATK3/T505XXU2ATK3/T505XXS2ATL1`
```

Скачайте последнюю версию прошивки и расшифруйте ее
```
samloader -m SM-T505 -r SER download -v T505XXS2ATL1/T505OXM2ATK3/T505XXU2ATK3/T505XXS2ATL1 -O .
samloader -m SM-T505 -r SER decrypt -v T505XXS2ATL1/T505OXM2ATK3/T505XXU2ATK3/T505XXS2ATL1 -i SM-T505_1_20201214142134_gx2y12bsp6_fac.zip.enc4 -o SM-T505_1_20201214142134_gx2y12bsp6_fac.zip
```

Распакуйте прошивку и отправьте файл, начинающийся на AP на устройство

```
unzip SM-T505_1_20201214142134_gx2y12bsp6_fac.zip
adb -d push AP_T505XXS2ATL1_CL20288130_QB36618241_REV00_user_low_ship_MULTI_CERT_meta_RKEY_OS10.tar.md5 /sdcard/
```

- Установите приложение **Magisk Manager**
```
adb install MagiskManager-v8.0.3.apk
```
- Пропатчите образ в приложении
- Получите модифицированный образ:
```
adb -d pull /sdcard/Download/magisk_patched.tar .
```
- Подключите планшет к ЭВМ с Windows, установите драйверы
- Запустите приложение **Odin** и убедитесь, что программа обнаружила устройство, находящееся в режиме **Download mode**. Если вы запускаете **Odin** в виртуальной машине (QEMU), необходимо
скачать последнюю версию и установить **virtio-win-guest-tools**.

:::info
Телефон/планшет можно держать в подключенном состоянии до начала установки не более ~5 минут, после - перезагружать, иначе могут быть проблемы с подключением через **Odin**.
:::

- Отметьте для установки соответствующие файлы таким образом, чтобы вместо образа `AP` был `magisk_patched.tar`. При этом раздел userdata можно не трогать.
- Загрузитесь в систему, проверьте, что у вас работает root-доступ.
- Включите режим для разработчиков и вновь разрешите отладку по usb.

#### Добавьте опцию для автоматического запуска adb по сети
```
adb shell
```
- разрешите root-доступ для оболочки:

```
mount -o remount,rw /
echo service.adb.tcp.port=5555 >> /system/build.prop
reboot
```
- Проверьте, что после после перезагрузки аппарат начинает слушать
tcp-порт и принимать соединения.

