# Решение проблем<br><br>

## Проблемы с подключением к роутеру по ssh
1. При выполнении `ssh root@192.168.1.1` выдает ошибку 
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
```
Решение: удалить файл `known_hosts` в папке C:/Users/<имя пользователя>/.ssh<br><br><br>

2. При выполнении `ssh root@192.168.1.1` выдает ошибку `Unable to negotiate with 192.168.1.1 port 22: no matching host key type found. Their offer: ssh-rsa`<br><br>
Решение: использовать для подключения команду `ssh -oHostKeyAlgorithms=+ssh-rsa root@192.168.1.1`<br><br><br>

## Проблемы с работой интернета
1. После перезагрузки не открывается ничего, кроме ВК и ориокса.<br><br>
Это происходит в случае, если на роутере выставлена неправильная [дата](/setup/wersonit_new.md#настройка-wi-fi), или если вы не выбрали тариф в личном кабинете OnPlus, а также, в более редких случаях, если у роутера задан неправильный [MAC-адрес](/setup/macaddr.md).<br><br><br>

2. При подключении компьютера к роутеру по проводу интернет отваливается. Но при подключении через Wi-Fi интернет есть и работает.<br><br>
Это происходит в случае, если MAC-адрес LAN-интерфейса роутера совпадает с MAC-адресом вашего Ethernet адаптера. Вам нужно вернуть LAN-интерфейсу роутера его родной [MAC-адрес](/setup/macaddr.md), чтобы эта проблема ушла.
<br><br><br>

3. После выполнения всех шагов по настройке и перезагрузки интернет не работает. Чтобы понять причину, нужно посмотреть логи авторизации.<br><br>
Для старых роутеров: Снова зайдите в роутер с помощью `ssh root@192.168.1.1`, введите пароль. После попадания в админку выполните команду `eth_name=$(uci show network.wan.device | awk -F"'" '{print $2}')`. После этого выполните `rm /var/run/wpa_supplicant/$eth_name`, а следом - `wpa_supplicant -D wired -i $eth_name -c /etc/config/wpa.conf`.<br><br>
Для новых роутеров: Перейдите в веб-интерфейсе на вкладку "Статус" -> "Системный журнал". В поле "Не □ включая:" введите wpa_supplicant.<br><br>
Дальнейшие действия зависят от того, какие сообщения выводятся. Вот такие могут быть варианты:<br><br>

> [!NOTE]  
> Во всех примерах вывода ниже WAN-интерфейс называется `eth0.2`. На некоторых новых роутерах он называется не `eth0.2`, а `wan`, а на старых - `eth1`. 

### Вариант 1<br>
В конце есть строки:
```
EAP-MSCHAPV2: Authentication succeeded
EAP-TLV: TLV Result - Success - EAP-TLV/Phase2 Completed
eth0.2: CTRL-EVENT-EAP-SUCCESS EAP authentication completed successfully
eth0.2: CTRL-EVENT-CONNECTED - Connection to 01:80:c2:00:00:03 completed [id=0 id_str=]
no PHY for ifname eth0.2
```

Если выводит вот такое сообщение, но интернет не работает - пишите в поддержку OnPlus. Проблема на их стороне. Не выдался автоматически ip-адрес. Ждите, пока выдадут. После этого роутер нужно перезагрузить. На старом роутере это делается нажатием `Ctrl+C` и выполнением команды `reboot`. На новом - через веб-интерфейс на вкладке "Система" -> "Перезагрузка".<br><br><br>

### Вариант 2<br>
Выводится вот это:
```
Successfully initialized wpa_supplicant
no PHY for ifname eth0.2
eth0.2: Associated with 01:80:c2:00:00:03
eth0.2: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
eth0.2: CTRL-EVENT-EAP-STARTED EAP authentication started
eth0.2: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=25
eth0.2: CTRL-EVENT-EAP-METHOD EAP vendor 0 method 25 (PEAP) selected
eth0.2: CTRL-EVENT-EAP-FAILURE EAP authentication failed
```
Либо же вот это:
```
Successfully initialized wpa_supplicant
no PHY for ifname eth0.2
eth0.2: Associated with 01:80:c2:00:00:03
eth0.2: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
eth0.2: CTRL-EVENT-EAP-STARTED EAP authentication started
eth0.2: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=25
eth0.2: CTRL-EVENT-EAP-METHOD EAP vendor 0 method 25 (PEAP) selected
eth0.2: CTRL-EVENT-EAP-PEER-CERT depth=0 subject='C=RU, ST=Zelenograd, O=MIET, OU=IAC, CN=campus.miet.ru/emailAddress=noc@miet.ru' hash=abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabca
eth0.2: CTRL-EVENT-EAP-PEER-CERT depth=1 subject='C=RU, ST=Zelenograd, L=Moscow, O=MIET, OU=CA, CN=CA ROOT/emailAddress=noc@miet.ru' hash=abcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabcabca
EAP-TLV: TLV Result - Failure
eth0.2: CTRL-EVENT-EAP-FAILURE EAP authentication failed
```
Если у вас именно так, то вы ввели неправильный логин или пароль от ориокса. Либо же вы забыли [поменять](/setup/macaddr.md) MAC-адрес WAN-интерфейса роутера.<br><br><br>

### Вариант 3<br>
Выводится вот это:
```
Successfully initialized wpa_supplicant
no PHY for ifname eth0.2
eth0.2: Associated with 01:80:c2:00:00:03
eth0.2: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
eth0.2: CTRL-EVENT-EAP-FAILURE EAP authentication failed
```
Если у вас именно так, то что-то не так с проводом, или, в очень редких случаях, с самим роутером. Проверьте, что провод от провайдера вставлен именно в WAN-разъем роутера. Иногда проводов бывает несколько. Проверьте, что взяли точно правильный. [Как проверить провод?](/setup/wire_check.md) Если ничего не помогло, то скорее всего, проблема на стороне миэта. Пишите в поддержку onplus, что у вас не работает авторизация, но работала раньше, они свяжутся с администрацией и починят вам всё в течение нескольких дней. Либо скажите, что рабочие что-то сломали, когда делали ремонт. Короче это как с чат-ботом, которого вы пытаетесь заставить позвать живого оператора.<br><br><br>

### Вариант 4<br>
Похоже на вариант 1, но последняя строчка вот такая:
```
EAP-TLV: TLV Result - Success - EAP-TLV/Phase2 Completed
```
А после нее нет никаких других строк. Если у вас именно так, то у вас проблема, решение которой я не знаю. Очень часто это происходит у иностранцев, у которых логин в ориоксе начинается с 10, а не с 8. Не знаю, с чем это связано. Возможно, у вас что-то не так с учетной записью. Попробуйте поменять пароль от ориокса и подождать пару дней. Если не поможет -  придется оформить интернет на соседа или на другого человека.

### Вариант 5<br>
Выводится ошибка: `unknown network field "eap"` или что-то подобное.
Это происходит потому, что установлена неправильная прошивка. Пишите мне, я помогу установить правильную.
