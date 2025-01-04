## Лабораторная работа - Реализация DHCPv6  

### Топология

![](/Lab8_2/Images/00_topology.jpg)

### 1. Таблица адресации 
Устройство | Интерфейс   | IPv6-адрес            |
-----------|-------------|-----------------------|
R1         | G0/0/0      | 2001:db8:acad:2::1/64 | 
|          |             | fe80::1               |
|          | G0/0/1      | 2001:db8:acad:1::1/64 | 
|          |             | fe80::1               |
R2         | G0/0/0      | 2001:db8:acad:2::2/64 |
|          |             | fe80::2               |
|          | G0/0/1      | 2001:db8:acad:3::1/64 |
|          |             | fe80::1               |
PC-A       | NIC         | DHCP                  |
PC-B       | NIC         | DHCP                  |

### Задачи

#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Проверка назначения адреса SLAAC от R1
#### Часть 3. Настройка и проверка сервера DHCPv6 без гражданства на R1
#### Часть 4. Настройка и проверка состояния DHCPv6 сервера на R1
#### Часть 5. Настройка и проверка DHCPv6 Relay на R2

### Общие сведения и сценарий

Динамическое назначение глобальных индивидуальных IPv6-адресов можно настроить тремя способами:

•	Автоматическая конфигурация адреса без сохранения состояния (Stateless Address Autoconfiguration, SLAAC)

•	DHCPv6 без отслеживания состояния

•	Адресация DHCPv6 с учетом состояний.

При использовании SLACC для назначения адресов IPv6 хостам сервер DHCPv6 не используется. Поскольку DHCPv6 сервер не используется при реализации SLACC, хосты не могут получать дополнительную важную сетевую информацию, включая адрес сервера доменных имен (DNS), а также имя домена.

При использовании Stateless DHCPv6 для назначения адресов IPv6 хосту сервер DHCPv6 используется для назначения дополнительной важной информации о сети, однако адрес IPv6 назначается с помощью SLACC.
При использовании DHCPv6 с отслеживанием состояния, сервер DHCP назначает всю информацию, включая IPv6-адрес узла.

Определение способа получения динамической IPv6-адресации зависит от установленных значений флагов, содержащихся в объявлениях маршрутизатора (сообщениях RA).

В предложенном сценарии размеры компании увеличились, и сетевые администраторы больше не имеют возможности назначать IP-адреса для устройств вручную. Ваша задача - настроить маршрутизатор R2 для назначения адресов IPv6 в двух разных подсетях, подключенных к маршрутизатору R1.

**Примечание:** Маршрутизаторы, используемые в практических лабораторных работах CCNA, - это Cisco 4221 с Cisco IOS XE Release 16.9.4 (образ universalk9). В лабораторных работах используются коммутаторы Cisco Catalyst 2960 с Cisco IOS версии 15.2(2) (образ lanbasek9). Можно использовать другие маршрутизаторы, коммутаторы и версии Cisco IOS. В зависимости от модели устройства и версии Cisco IOS доступные команды и результаты их выполнения могут отличаться от тех, которые показаны в лабораторных работах. Правильные идентификаторы интерфейса см. в сводной таблице по интерфейсам маршрутизаторов в конце лабораторной работы.

**Примечание:** Убедитесь, что у всех маршрутизаторов и коммутаторов была удалена начальная конфигурация. Если вы не уверены в этом, обратитесь к инструктору.

### Необходимые ресурсы
•	2 маршрутизатора (Cisco 4221 с универсальным образом Cisco IOS XE версии 16.9.4 или аналогичным)

•	2 коммутатора (Cisco 2960 с образом lanbasek9 Cisco IOS версии 15.2 (2) или аналогичным) - опционально

•	2 ПК (ОС Windows с программой эмуляции терминалов, такой как Tera Term)

•	Консольные кабели для настройки устройств Cisco IOS через консольные порты.

•	Кабели Ethernet, расположенные в соответствии с топологией

### Инструкции
### Часть 1. Создание сети и настройка основных параметров устройства

В первой части лабораторной работы вам предстоит создать топологию сети и настроить базовые параметры для узлов ПК и коммутаторов.

#### Шаг 1. Создайте сеть согласно топологии.

Подключите устройства, как показано в топологии, и подсоедините необходимые кабели.

#### Шаг 2. Настройте базовые параметры каждого коммутатора. (необязательно)

Откройте окно конфигурации

a.	Присвойте коммутатору имя устройства.

b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

f.	Зашифруйте открытые пароли.

g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.

h.	Отключите все неиспользуемые порты.

i.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.
```
Switch(config)#hostname S1
S1(config)#no ip domain-lookup 
S1(config)#enable secret class
S1(config)#line 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#exit
S1(config)#service password-encryption 
S1(config)#banner motd $
Enter TEXT message.  End with the character '$'.
ATTENTION!!! Unauthorized access is strictly progibited!!! $

S1(config)#
S1(config)#interface range f0/1-4, f0/7-24, g0/1-2
S1(config-if-range)#shutdown
S1(config-if-range)#end
S1#
%SYS-5-CONFIG_I: Configured from console by console

S1#
S1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
S1#
```
```
Switch(config)#hostname S2
S2(config)#no ip domain-lookup 
S2(config)#enable secret class
S2(config)#line 0
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#line vty 0 4
S2(config-line)#password cisco
S2(config-line)#login
S2(config-line)#exit
S2(config)#service password-encryption 
S2(config)#banner motd $
Enter TEXT message.  End with the character '$'.

ATTENTION!!! Unauthorized access is strictly prohibited!!! $

S2(config)#
S2(config)#interface range f0/1-4, f0/7-17, f0/19-24, g0/1-2
S2(config-if-range)#shutdown
S2(config-if-range)#end
S2#
%SYS-5-CONFIG_I: Configured from console by console

S2#
S2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
S2#
```
Закройте окно настройки.

#### Шаг 3. Произведите базовую настройку маршрутизаторов.

Откройте окно конфигурации

a.	Назначьте маршрутизатору имя устройства.

b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

f.	Зашифруйте открытые пароли.

g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.

h.	Активация IPv6-маршрутизации

i.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

```
Router>en
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R1
R1(config)#no ip domain-lookup
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
R1(config)#service password-encryption
R1(config)#banner MOTD $
Enter TEXT message.  End with the character '$'.
ATTENTION!!! Unauthorized access is strictly prohibited!!! $

R1(config)#
R1(config)#ipv6 unicast-routing 
R1(config)#exit
R1#
%SYS-5-CONFIG_I: Configured from console by console

R1#
R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
R1#
```
```
Router>en
Router#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#hostname R2
R2(config)#no ip domain-lookup
R2(config)#enable secret class
R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#exit
R2(config)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#exit
R2(config)#service password-encryption 
R2(config)#banner motd $
Enter TEXT message.  End with the character '$'.
ATTENTION!!! Unauthorized access is strictly prohibited!!! $

R2(config)#ipv6 unicast-routing 
R2(config)#exit
R2#
%SYS-5-CONFIG_I: Configured from console by console

R2#
R2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
R2#
```

#### Шаг 4. Настройка интерфейсов и маршрутизации для обоих маршрутизаторов.

a.	Настройте интерфейсы G0/0/0 и G0/1 на R1 и R2 с адресами IPv6, указанными в таблице выше.
```
R1(config)#interface g0/0/1
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#ipv6 address 2001:db8:acad:1::1/64
R1(config-if)#no shutdown

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R1(config-if)#interface g0/0/0
R1(config-if)#ipv6 address fe80::1 link-local
R1(config-if)#ipv6 address 2001:db8:acad:2::1/64
R1(config-if)#no shutdown

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

R1(config-if)#
R1(config-if)#end
R1#
%SYS-5-CONFIG_I: Configured from console by console
```
```
R2(config)#interface g0/0/1
R2(config-if)#ipv6 address fe80::1 link-local
R2(config-if)#ipv6 address 2001:db8:acad:3::1/64
R2(config-if)#no shutdown

R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R2(config-if)#
R2(config-if)#interface g0/0/0
R2(config-if)#ipv6 address fe80::2 link-local
R2(config-if)#ipv6 address 2001:db8:acad:2::2/64
R2(config-if)#no shutdown

R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up

R2(config-if)#end
R2#
%SYS-5-CONFIG_I: Configured from console by console
```

b.	Настройте маршрут по умолчанию на каждом маршрутизаторе, который указывает на IP-адрес G0/0/0 на другом маршрутизаторе.

c.	Убедитесь, что маршрутизация работает с помощью пинга адреса G0/0/1 R2 из R1

d.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

### Часть 2. Проверка назначения адреса SLAAC от R1

В части 2 вы убедитесь, что узел PC-A получает адрес IPv6 с помощью метода SLAAC.
Включите PC-A и убедитесь, что сетевой адаптер настроен для автоматической настройки IPv6.
Через несколько минут результаты команды ipconfig должны показать, что PC-A присвоил себе адрес из сети 2001:db8:1::/64.

`C:\Users\Student> ipconfig `

Настройка IP для Windows

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::20B:BEFF:FEA8:B2D9
   IPv6 Address....................: 2001:DB8:ACAD:1:20B:BEFF:FEA8:B2D9
   Autoconfiguration IPv4 Address..: 169.254.178.217
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

**Вопрос:**
Откуда взялась часть адреса с идентификатором хоста?

**Ответ:**

Хост генерирует адрес на основе MAC-адреса

### Часть 3. Настройка и проверка сервера DHCPv6 на R1

В части 3 выполняется настройка и проверка состояния DHCP-сервера на R1. Цель состоит в том, чтобы предоставить PC-A информацию о DNS-сервере и домене.

#### Шаг 1. Более подробно изучите конфигурацию PC-A.

a.	Выполните команду `ipconfig /all` на PC-A и посмотрите на результат.

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 000B.BEA8.B2D9
   Link-local IPv6 Address.........: FE80::20B:BEFF:FEA8:B2D9
   IPv6 Address....................: 2001:DB8:ACAD:1:20B:BEFF:FEA8:B2D9
   Autoconfiguration IP Address....: 169.254.178.217
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-27-27-A6-90-00-0B-BE-A8-B2-D9
   DNS Servers.....................: ::
                                     0.0.0.0
```

b.	Обратите внимание, что основной DNS-суффикс отсутствует. Также обратите внимание, что предоставленные адреса DNS-сервера являются адресами «локального сайта anycast», а не одноадресные адреса, как ожидалось.

#### Шаг 2. Настройте R1 для предоставления DHCPv6 без состояния для PC-A.

a.	Создайте пул DHCP IPv6 на R1 с именем R1-STATELESS. В составе этого пула назначьте адрес DNS-сервера как 2001:db8:acad: :1, а имя домена — как stateless.com.

Откройте окно конфигурации
```
R1(config)# ipv6 dhcp pool R1-STATELESS
R1(config-dhcp)# dns-server 2001:db8:acad::254
R1(config-dhcp)# domain-name STATELESS.com
```

b.	Настройте интерфейс G0/0/1 на R1, чтобы предоставить флаг конфигурации OTHER для локальной сети R1 и укажите только что созданный пул DHCP в качестве ресурса DHCP для этого интерфейса.
```
R1(config)# interface g0/0/1
R1(config-if)# ipv6 nd other-config-flag 
R1(config-if)# ipv6 dhcp server R1-STATELESS
```
c.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.
d.	Перезапустите PC-A.
e.	Проверьте вывод `ipconfig /all` и обратите внимание на изменения.

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: STATELESS.com 
   Physical Address................: 000B.BEA8.B2D9
   Link-local IPv6 Address.........: FE80::20B:BEFF:FEA8:B2D9
   IPv6 Address....................: 2001:DB8:ACAD:1:20B:BEFF:FEA8:B2D9
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 957219645
   DHCPv6 Client DUID..............: 00-01-00-01-27-27-A6-90-00-0B-BE-A8-B2-D9
   DNS Servers.....................: 2001:DB8:ACAD::254
                                     0.0.0.0
```
f.	Тестирование подключения с помощью пинга IP-адреса интерфейса G0/1 R2.
```
C:\>ping 2001:db8:acad:3::1

Pinging 2001:db8:acad:3::1 with 32 bytes of data:

Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254
Reply from 2001:DB8:ACAD:3::1: bytes=32 time<1ms TTL=254

Ping statistics for 2001:DB8:ACAD:3::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
### Часть 4. Настройка сервера DHCPv6 с сохранением состояния на R1

В части 4 настраивается R1 для ответа на запросы DHCPv6 от локальной сети на R2.

a.	Создайте пул DHCPv6 на R1 для сети 2001:db8:acad:3:aaa::/80. Это предоставит адреса локальной сети, подключенной к интерфейсу G0/0/1 на R2. В составе пула задайте DNS-сервер 2001:db8:acad: :254 и задайте доменное имя STATEFUL.com.

Откройте окно конфигурации

```
R1(config)# ipv6 dhcp pool R2-STATEFUL
R1(config-dhcp)# address prefix 2001:db8:acad:3:aaa::/80
R1(config-dhcp)# dns-server 2001:db8:acad::254
R1(config-dhcp)# domain-name STATEFUL.com
```
b.	Назначьте только что созданный пул DHCPv6 интерфейсу g0/0/0 на R1.
```
R1(config)# interface g0/0/0
R1(config-if)# ipv6 dhcp server R2-STATEFUL
```

### Часть 5. Настройка и проверка ретрансляции DHCPv6 на R2.

В части 5 необходимо настроить и проверить ретрансляцию DHCPv6 на R2, позволяя PC-B получать адрес IPv6.

#### аг 1. Включите PC-B и проверьте адрес SLAAC, который он генерирует.

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 00E0.B004.62A2
   Link-local IPv6 Address.........: FE80::2E0:B0FF:FE04:62A2
   IPv6 Address....................: 2001:DB8:ACAD:3:2E0:B0FF:FE04:62A2
   Autoconfiguration IP Address....: 169.254.98.162
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-13-01-8E-57-00-E0-B0-04-62-A2
   DNS Servers.....................: ::
                                     0.0.0.0
```

Обратите внимание на вывод, что используется префикс 2001:db8:acad:3::

#### Шаг 2. Настройте R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1.

a.	Настройте команду ipv6 dhcp relay на интерфейсе R2 G0/0/1, указав адрес назначения интерфейса G0/0/0 на R1. Также настройте команду managed-config-flag .

Откройте окно конфигурации

```
R2(config) # интерфейс g0/0/1
R2(config-if)# ipv6 nd managed-config-flag
R2(config-if)# ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0
```

Команда `ipv6 dhcp relay` не сработала:
```
R2(config-if)#ipv6 dhcp relay destination 2001:db8:acad:2::1 g0/0/0
                        ^
% Invalid input detected at '^' marker.
```

b.	Сохраните конфигурацию.

Закройте окно настройки.

#### Шаг 3. Попытка получить адрес IPv6 из DHCPv6 на PC-B.

a.	Перезапустите PC-B.

b.	Откройте командную строку на PC-B и выполните команду ipconfig /all и проверьте выходные данные, чтобы увидеть результаты операции ретрансляции DHCPv6.

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Physical Address................: 00E0.B004.62A2
   Link-local IPv6 Address.........: FE80::2E0:B0FF:FE04:62A2
   IPv6 Address....................: ::
   Autoconfiguration IP Address....: 169.254.98.162
   Subnet Mask.....................: 255.255.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
   DHCP Servers....................: 0.0.0.0
   DHCPv6 IAID.....................: 1186849477
   DHCPv6 Client DUID..............: 00-01-00-01-13-01-8E-57-00-E0-B0-04-62-A2
   DNS Servers.....................: ::
                                     0.0.0.0

```
c.	Проверьте подключение с помощью пинга IP-адреса интерфейса R0 G0/0/1.
