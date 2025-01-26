## Лабораторная работа - Настройка NAT для IPv4

### Топология

![](/Lab12/Images/00_topology.jpg)

### Таблица адресации 

Устройство | Интерфейс   | IP-адрес        | Маска подсети   |
-----------|-------------|-----------------|-----------------|
R1         | G0/0/0      | 209.165.200.230 | 255.255.255.248 |
|          | G0/0/1      | 192.168.1.1     | 255.255.255.0   |
R2         | G0/0/0      | 209.165.200.225 | 255.255.255.248 |
|          | Lo1         | 209.165.200.1   | 255.255.255.224 |
S1         | VLAN 1      | 192.168.1.11    | 255.255.255.0   |
S2         | VLAN 1      | 192.168.1.12    | 255.255.255.0   |
PC-A       | NIC         | 192.168.1.2     | 255.255.255.0   |
PC-B       | NIC         | 192.168.1.3     | 255.255.255.0   |

### Цели
#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Настройка и проверка NAT для IPv4
#### Часть 3. Настройка и проверка PAT для IPv4
#### Часть 4. Настройка и проверка статического NAT для IPv4.

### Общие сведения/сценарий

Преобразование (NAT) — это процесс, при котором сетевое устройство, например маршрутизатор Cisco, назначает публичный адрес узлам в пределах частной сети. NAT используют для сокращения количества публичных IP-адресов, используемых организацией, поскольку количество доступных публичных IPv4-адресов ограничено.

Интернет-провайдер выделил компании общедоступное пространство IP-адресов 209.165.200.224/29. Эта сеть используется для обращения к каналу между маршрутизатором ISP (R2) и шлюзом компании (R1). Первый адрес (209.165.200.225) назначается интерфейсу g0/0 на R2, а последний адрес (209.165.200.230) назначается интерфейсу g0/0/0 на R1. Остальные адреса (209.165.200.226-209.165.200.229) будут использоваться для предоставления доступа в Интернет хостам компании. Маршрут по умолчанию используется от R1 до R2. Подключение интернет-провайдера к Интернету смоделировано loopback-адресом на маршрутизаторе интернет-провайдера.

В этой лабораторной работе  вы будете настраивать различные типы NAT. Вы выполните тестирование, отображение и проверку осуществления всех преобразований и проанализируете статистику NAT/PAT для контроля процесса.

**Примечание:** Маршрутизаторы, используемые в практических лабораторных работах CCNA, - это Cisco 4221 с Cisco IOS XE Release 16.9.3 (образ universalk9). В лабораторных работах используются коммутаторы Cisco Catalyst 2960 с Cisco IOS версии 15.2(2) (образ lanbasek9). Можно использовать другие маршрутизаторы, коммутаторы и версии Cisco IOS. В зависимости от модели устройства и версии Cisco IOS доступные команды и результаты их выполнения могут отличаться от тех, которые показаны в лабораторных работах. Правильные идентификаторы интерфейса см. в сводной таблице по интерфейсам маршрутизаторов в конце лабораторной работы.

**Примечание:** Убедитесь, что у всех маршрутизаторов и коммутаторов была удалена начальная конфигурация. Если вы не уверены в этом, обратитесь к инструктору.

### Необходимые ресурсы
+	2 маршрутизатора (Cisco 4221 с универсальным образом Cisco IOS XE версии 16.9.4 или аналогичным)
+	2 коммутатора (Cisco 2960 с операционной системой Cisco IOS 15.2(2) (образ lanbasek9) или аналогичная модель)
+	2 ПК (ОС Windows с программой эмуляции терминалов, такой как Tera Term)
+	Консольные кабели для настройки устройств Cisco IOS через консольные порты.
+	Кабели Ethernet, расположенные в соответствии с топологией

### Инструкции

### Часть 1. Создание сети и настройка основных параметров устройства

В первой части лабораторной работы вам предстоит создать топологию сети и настроить базовые параметры для узлов ПК и коммутаторов.

#### Шаг 1. Подключите кабели сети согласно приведенной топологии.

Подключите устройства в соответствии с топологией и подсоедините соответствующие кабели.

#### Шаг 2. Произведите базовую настройку маршрутизаторов.

Откройте окно конфигурации

a.	Назначьте маршрутизатору имя устройства.

`Router(config)#hostname R1`

`Router(config)#hostname R2`

b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

`R1(config)#no ip domain-lookup`

`R2(config)#no ip domain lookup`

c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.

`R1(config)#enable secret class`

`R2(config)#enable secret class`

d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

```
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
```

```
R2(config)#line console 0
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#exit
```

e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

```
R1(config)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#exit
```

```
R2(config-line)#line vty 0 4
R2(config-line)#password cisco
R2(config-line)#login
R2(config-line)#exit
```

f.	Зашифруйте открытые пароли.

`R1(config)#service password-encryption `

`R2(config)#service password-encryption `

g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.

```
R1(config)#banner motd $
Enter TEXT message.  End with the character '$'.
ATTINTION!!! Untauthorized access is strictly prohibited! $

R1(config)#
```

```
R2(config)#banner motd $
Enter TEXT message.  End with the character '$'.
ATTENTION!!! Unauthorized access is strictly prohibited! $

R2(config)#
```

h.	Настройте IP-адресации интерфейса, как указано в таблице выше.

```
R1(config-if)#ip add
R1(config-if)#ip address 209.165.200.230 255.255.255.248
R1(config-if)#no sh

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

R1(config-if)#
R1(config-if)#
R1(config-if)#exit
R1(config)#
R1(config)#
R1(config)#interface g0/0/1
R1(config-if)#
R1(config-if)#
R1(config-if)#ip add
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#
R1(config-if)#
R1(config-if)#no shut

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R1(config-if)#
R1(config-if)#
R1(config-if)#exit
```

```
R2(config)#Interface g0/0/0
R2(config-if)#
R2(config-if)#
R2(config-if)#ip address 209.165.200.255 255.255.255.248
Bad mask /29 for address 209.165.200.255
R2(config-if)#
R2(config-if)#ip address 209.165.200.225 255.255.255.248
R2(config-if)#
R2(config-if)#
R2(config-if)#no shut

R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up

R2(config-if)#
R2(config-if)#interface Loopback1

R2(config-if)#
%LINK-5-CHANGED: Interface Loopback1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Loopback1, changed state to up

R2(config-if)#
R2(config-if)#
R2(config-if)#ip address 209.165.200.1 255.255.255.224
R2(config-if)#
R2(config-if)#no shut
R2(config-if)#
R2(config-if)#
R2(config-if)#
R2(config-if)#end
```

i.	Настройте маршрут по умолчанию. от R2 до  R1.

```
R1(config)#ip route 0.0.0.0 0.0.0.0 209.165.200.225
R1(config)#exit
```

j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

```
R1#show startup-config 
Using 915 bytes
!
version 16.6.4
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname R1
!
!
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
!
!
!
!
!
!
no ip domain-lookup
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface GigabitEthernet0/0/0
 ip address 209.165.200.230 255.255.255.248
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/1
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
ip route 0.0.0.0 0.0.0.0 209.165.200.225 
!
ip flow-export version 9
!
!
!
banner motd ^C
ATTINTION!!! Untauthorized access is strictly prohibited! ^C
!
!
!
!
!
line con 0
 password 7 0822455D0A16
 login
!
line aux 0
!
line vty 0 4
 password 7 0822455D0A16
 login
!
!
!
end

```

```
R2#show startup-config 
Using 923 bytes
!
version 16.6.4
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname R2
!
!
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
!
!
!
!
!
!
ip cef
no ipv6 cef
!
!
!
!
!
!
!
!
!
!
no ip domain-lookup
!
!
spanning-tree mode pvst
!
!
!
!
!
!
interface Loopback1
 ip address 209.165.200.1 255.255.255.224
!
interface GigabitEthernet0/0/0
 ip address 209.165.200.225 255.255.255.248
 duplex auto
 speed auto
!
interface GigabitEthernet0/0/1
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface GigabitEthernet0/0/2
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface Vlan1
 no ip address
 shutdown
!
ip classless
!
ip flow-export version 9
!
!
!
banner motd ^C
ATTENTION!!! Unauthorized access is strictly prohibited! ^C
!
!
!
!
!
line con 0
 password 7 0822455D0A16
 login
!
line aux 0
!
line vty 0 4
 password 7 0822455D0A16
 login
!
!
!
end
```

Закройте окно настройки.

#### Шаг 3. Настройте базовые параметры каждого коммутатора.

Откройте окно конфигурации

a.	Присвойте коммутатору имя устройства.

```
Switch(config)#hostname S1
```

```
Switch(config)#hostname S2
```

b.	Отключите поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

```
S1(config)#no ip domain-lookup 
```

```
S2(config)#no ip domain-lookup
```

c.	Назначьте class в качестве зашифрованного пароля привилегированного режима EXEC.
```
S1(config)#enable secret class
```

```
S2(config)#enable secret class
```

d.	Назначьте cisco в качестве пароля консоли и включите вход в систему по паролю.

```
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
```

```
S2(config)#line console 0
S2(config-line)#password cisco
S2(config-line)#login
```
e.	Назначьте cisco в качестве пароля VTY и включите вход в систему по паролю.

```
S1(config)#line vty 0 15
S1(config-line)#password cisco
S1(config-line)#login
```

```
S2(config)#line vty 0 15
S2(config-line)#password cisco
S2(config-line)#login
```
f.	Зашифруйте открытые пароли.

```
S1(config)#service password-encryption 
```

```
S2(config)#service password-encryption 
```

g.	Создайте баннер с предупреждением о запрете несанкционированного доступа к устройству.

```
S1(config)#banner motd $
Enter TEXT message.  End with the character '$'.
ATTENTION!!! Unauthorized access is strictly prohibited!!! $

S1(config)#
```

```
S2(config)#banner motd $
Enter TEXT message.  End with the character '$'.
ATTENTION!!! Unauthorized access is strictly prohibited!!! $

S2(config)#
```

h.	Выключите все интерфейсы, которые не будут использоваться.

```
S1(config)#interface range f0/2-4, f0/7-24, g0/1-2
S1(config-if-range)#shutdown
```

```
S2(config)#interface range f0/2-17, f0/19-24, g0/1-2
S2(config-if-range)#shutdown
```

i.	Настройте IP-адресации интерфейса, как указано в таблице выше.

```
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.11 255.255.255.0
S1(config-if)#no shutdown

S1(config-if)#
%LINK-5-CHANGED: Interface Vlan1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

S1(config-if)#
S1(config-if)#exit
S1(config)#ip default-gateway 192.168.1.1
```

```
S2(config)#interface vlan 1
S2(config-if)#ip address 192.168.1.12 255.255.255.0
S2(config-if)#no shutdown

S2(config-if)#
%LINK-5-CHANGED: Interface Vlan1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

S2(config)#
S2(config)#exit
S2(config)#ip default-gateway 192.168.1.1
```

j.	Сохраните текущую конфигурацию в файл загрузочной конфигурации.

```
S1#show startup-config 
Using 1576 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname S1
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
!
!
!
no ip domain-lookup
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
!
interface FastEthernet0/2
 shutdown
!
interface FastEthernet0/3
 shutdown
!
interface FastEthernet0/4
 shutdown
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
 shutdown
!
interface FastEthernet0/8
 shutdown
!
interface FastEthernet0/9
 shutdown
!
interface FastEthernet0/10
 shutdown
!
interface FastEthernet0/11
 shutdown
!
interface FastEthernet0/12
 shutdown
!
interface FastEthernet0/13
 shutdown
!
interface FastEthernet0/14
 shutdown
!
interface FastEthernet0/15
 shutdown
!
interface FastEthernet0/16
 shutdown
!
interface FastEthernet0/17
 shutdown
!
interface FastEthernet0/18
 shutdown
!
interface FastEthernet0/19
 shutdown
!
interface FastEthernet0/20
 shutdown
!
interface FastEthernet0/21
 shutdown
!
interface FastEthernet0/22
 shutdown
!
interface FastEthernet0/23
 shutdown
!
interface FastEthernet0/24
 shutdown
!
interface GigabitEthernet0/1
 shutdown
!
interface GigabitEthernet0/2
 shutdown
!
interface Vlan1
 ip address 192.168.1.11 255.255.255.0
!
ip default-gateway 192.168.1.1
!
banner motd ^C
ATTENTION!!! Unauthorized access is strictly prohibited!!! ^C
!
!
!
line con 0
 password 7 0822455D0A16
 login
!
line vty 0 4
 password 7 0822455D0A16
 login
line vty 5 15
 password 7 0822455D0A16
 login
!
!
!
!
end
```
```
S2#show startup-config 
Using 1590 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname S2
!
enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
!
!
!
no ip domain-lookup
!
!
!
spanning-tree mode pvst
spanning-tree extend system-id
!
interface FastEthernet0/1
!
interface FastEthernet0/2
 shutdown
!
interface FastEthernet0/3
 shutdown
!
interface FastEthernet0/4
 shutdown
!
interface FastEthernet0/5
 shutdown
!
interface FastEthernet0/6
 shutdown
!
interface FastEthernet0/7
 shutdown
!
interface FastEthernet0/8
 shutdown
!
interface FastEthernet0/9
 shutdown
!
interface FastEthernet0/10
 shutdown
!
interface FastEthernet0/11
 shutdown
!
interface FastEthernet0/12
 shutdown
!
interface FastEthernet0/13
 shutdown
!
interface FastEthernet0/14
 shutdown
!
interface FastEthernet0/15
 shutdown
!
interface FastEthernet0/16
 shutdown
!
interface FastEthernet0/17
 shutdown
!
interface FastEthernet0/18
!
interface FastEthernet0/19
 shutdown
!
interface FastEthernet0/20
 shutdown
!
interface FastEthernet0/21
 shutdown
!
interface FastEthernet0/22
 shutdown
!
interface FastEthernet0/23
 shutdown
!
interface FastEthernet0/24
 shutdown
!
interface GigabitEthernet0/1
 shutdown
!
interface GigabitEthernet0/2
 shutdown
!
interface Vlan1
 ip address 192.168.1.12 255.255.255.0
!
ip default-gateway 192.168.1.1
!
banner motd ^C

ATTENTION!!! Unauthorized access is stric.tly prohibited!!! 

^C
!
!
!
line con 0
 password 7 0822455D0A16
 login
!
line vty 0 4
 password 7 0822455D0A16
 login
line vty 5 15
 password 7 0822455D0A16
 login
!
!
!
!
end
```

### Часть 2. Настройка и проверка NAT для IPv4.

В части 2 необходимо настроить и проверить NAT для IPv4.

#### Шаг 1. Настройте NAT на R1, используя пул из трех адресов 209.165.200.226-209.165.200.228. 

Откройте окно конфигурации

a.	Настройте простой список доступа, который определяет, какие хосты будут разрешены для трансляции. В этом случае все устройства в локальной сети R1 имеют право на трансляцию.

`R1(config)# access-list 1 permit 192.168.1.0 0.0.0.255 `

b.	Создайте пул NAT и укажите ему имя и диапазон используемых адресов.

`R1(config)# ip nat pool PUBLIC_ACCESS 209.165.200.226 209.165.200.228 netmask 255.255.255.248 `

**Примечание:** Параметр маски сети не является разделителем IP-адресов. Это должна быть правильная маска подсети для назначенных адресов, даже если вы используете не все адреса подсети в пуле. 

c.	Настройте перевод, связывая ACL и пул с процессом преобразования.

`R1(config)# ip nat inside source list 1 pool PUBLIC_ACCESS `

Примечание: Три очень важных момента. Во-первых, слово «inside» имеет решающее значение для работы такого рода NAT. Если вы опустить его, NAT не будет работать. Во-вторых, номер списка — это номер ACL, настроенный на предыдущем шаге. В-третьих, имя пула чувствительно к регистру. 

d.	Задайте внутренний (inside) интерфейс. 
```
R1(config)# interface g0/0/1
R1(config-if)# ip nat inside
```

e.	Определите внешний (outside) интерфейс.
```
R1(config)# interface g0/0/0
R1(config-if)# ip nat outside
```
#### Шаг 2. Проверьте и проверьте конфигурацию.

a.	С PC-B,  запустите эхо-запрос интерфейса Lo1 (209.165.200.1) на R2. Если эхо-запрос не прошел, выполните процес поиска и устранения неполадок. На R1 отобразите таблицу NAT на R1 с помощью команды `show ip nat translations`.

```
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Request timed out.
Request timed out.
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 2, Lost = 2 (50% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```


```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:1 192.168.1.3:1      209.165.200.1:1    209.165.200.1:1
icmp 209.165.200.226:2 192.168.1.3:2      209.165.200.1:2    209.165.200.1:2
icmp 209.165.200.226:3 192.168.1.3:3      209.165.200.1:3    209.165.200.1:3
icmp 209.165.200.226:4 192.168.1.3:4      209.165.200.1:4    209.165.200.1:4
```

### Вопросы:
1) Во что был транслирован внутренний локальный адрес PC-B?

На 209.165.200.226

2) Какой тип адреса NAT является переведенным адресом?

Глобальный внутренний


b.	С PC-A, запустите  эхо-запрос интерфейса Lo1 (209.165.200.1) на R2. Если эхо-запрос не прошел, выполните отладку. На R1 отобразите таблицу NAT на R1 с помощью команды `show ip nat translations`.

```
C:\>ping 209.165.200.1

Pinging 209.165.200.1 with 32 bytes of data:

Reply from 209.165.200.1: bytes=32 time=2ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time<1ms TTL=254
Reply from 209.165.200.1: bytes=32 time=2ms TTL=254

Ping statistics for 209.165.200.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 2ms, Average = 1ms
```


```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:5 192.168.1.2:5      209.165.200.1:5    209.165.200.1:5
icmp 209.165.200.226:6 192.168.1.2:6      209.165.200.1:6    209.165.200.1:6
icmp 209.165.200.226:7 192.168.1.2:7      209.165.200.1:7    209.165.200.1:7
icmp 209.165.200.226:8 192.168.1.2:8      209.165.200.1:8    209.165.200.1:8
```

c.	Обратите внимание, что предыдущая трансляция для PC-B все еще находится в таблице. Из S1, эхо-запрос интерфейса Lo1 (209.165.200.1) на R2. Если эхо-запрос не прошел, выполните отладку. На R1 отобразите таблицу NAT на R1 с помощью команды `show ip nat translations`.

```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:10192.168.1.2:10     209.165.200.1:10   209.165.200.1:10
icmp 209.165.200.226:11192.168.1.2:11     209.165.200.1:11   209.165.200.1:11
icmp 209.165.200.226:12192.168.1.2:12     209.165.200.1:12   209.165.200.1:12
icmp 209.165.200.226:9 192.168.1.2:9      209.165.200.1:9    209.165.200.1:9
```

d.	Теперь запускаем пинг R2 Lo1 из S2. На этот раз перевод завершается неудачей, и вы получаете эти сообщения (или аналогичные) на консоли R1:

```
Jan 26 13:28:31.432: %IOSXE-6-PLATFORM: R0/0: cpp_cp: QFP:0.0 Thread:000 TS:00000001473688385900 %NAT-6-ADDR_ALLOC_FAILURE: Address allocation failed; pool 1 may be exhausted [2]
```

e.	Это ожидаемый результат, потому что выделено только 3 адреса, и мы попытались ping Lo1 с четырех устройств. Напомним, что NAT — это трансляция «один-в-один». Как много выделено трансляций? Введите команду show ip nat translations verbose , и вы увидите, что ответ будет 24 часа.

```
R1#show ip nat translations verbose
                            ^
% Invalid input detected at '^' marker.
```

f.	Учитывая, что пул ограничен тремя адресами, NAT для пула адресов недостаточно для нашего приложения. Очистите преобразование NAT и статистику, и мы перейдем к PAT.

```
R1# clear ip nat translations * 
R1# clear ip nat statistics 
```

Закройте окно настройки.

### Часть 3. Настройка и проверка PAT для IPv4.

В части 3 необходимо настроить замену NAT на PAT в пул адресов, а затем на PAT с помощью интерфейса.

#### Шаг 1. Удалите команду преобразования на R1.

Откройте окно конфигурации

Компоненты конфигурации преобразования адресов в основном одинаковы; что-то (список доступа) для идентификации адресов, пригодных для перевода, дополнительно настроенный пул адресов для их преобразования и команды, необходимые для идентификации внутреннего и внешнего интерфейсов. Из части 1 наш список доступа (список доступа 1) по-прежнему корректен для сетевого сценария, поэтому нет необходимости воссоздавать его. Мы будем использовать один и тот же пул адресов, поэтому нет необходимости воссоздавать эту конфигурацию. Кроме того, внутренний и внешний интерфейсы не меняются. Чтобы начать работу в части 3, удалите команду, связывающую ACL и пул вместе.

`R1(config)# no ip nat inside source list 1 pool PUBLIC_ACCESS `

#### Шаг 2. Добавьте команду PAT на R1.

Теперь настройте преобразование PAT в пул адресов (помните, что ACL и Pool уже настроены, так что это единственная команда, которую нам нужно изменить с NAT на PAT).

`R1(config)# ip nat inside source list 1 pool PUBLIC_ACCESS overload `

#### Шаг 3. Протестируйте и проверьте конфигурацию.

a.	Давайте проверим, что PAT работает. С PC-B,  запустите эхо-запрос интерфейса Lo1 (209.165.200.1) на R2. Если эхо-запрос не прошел, выполните отладку. На R1 отобразите таблицу NAT на R1 с помощью команды `show ip nat translations`.

```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:1 192.168.1.3:1      209.165.200.1:1    209.165.200.1:1
icmp 209.165.200.226:2 192.168.1.3:2      209.165.200.1:2    209.165.200.1:2
icmp 209.165.200.226:3 192.168.1.3:3      209.165.200.1:3    209.165.200.1:3
icmp 209.165.200.226:4 192.168.1.3:4      209.165.200.1:4    209.165.200.1:4
```

#### Вопросы:
1) Во что был транслирован внутренний локальный адрес PC-B?

На 209.165.200.226

2) Какой тип адреса NAT является переведенным адресом?

 Глобальный внутренний

 Чем отличаются выходные данные команды `show ip nat translations` из упражнения NAT?
Никакого специального перевода между внутренним и внешним указанными адресами нет.
 
b.	С PC-A, запустите эхо-запрос интерфейса Lo1 (209.165.200.1) на R2. Если эхо-запрос не прошел, выполните отладку. На R1 отобразите таблицу NAT на R1 с помощью команды `show ip nat translations`.

```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:1 192.168.1.2:1      209.165.200.1:1    209.165.200.1:1
icmp 209.165.200.226:2 192.168.1.2:2      209.165.200.1:2    209.165.200.1:2
icmp 209.165.200.226:3 192.168.1.2:3      209.165.200.1:3    209.165.200.1:3
icmp 209.165.200.226:4 192.168.1.2:4      209.165.200.1:4    209.165.200.1:4
```

Обратите внимание, что есть только одна трансляция. Отправьте ping еще раз, и быстро вернитесь к маршрутизатору и введите команду `show ip nat translations verbose` , и вы увидите, что произошло.

```
R1#show ip nat translations verbose
                            ^
% Invalid input detected at '^' marker.
```

Как вы можете видеть, время ожидания перевода было отменено с 24 часов до 1 минуты.

c.	Генерирует трафик с нескольких устройств для наблюдения PAT. На PC-A и PC-B используйте параметр -t с командой ping, чтобы отправить безостановочный ping на интерфейс Lo1 R2 (ping -t 209.165.200.1), затем вернитесь к R1 и выполните команду `show ip nat translations`:

```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.226:1024192.168.1.2:9      209.165.200.1:9    209.165.200.1:1024
icmp 209.165.200.226:1025192.168.1.2:10     209.165.200.1:10   209.165.200.1:1025
icmp 209.165.200.226:1026192.168.1.2:11     209.165.200.1:11   209.165.200.1:1026
icmp 209.165.200.226:1027192.168.1.2:12     209.165.200.1:12   209.165.200.1:1027
icmp 209.165.200.226:1028192.168.1.2:13     209.165.200.1:13   209.165.200.1:1028
icmp 209.165.200.226:1029192.168.1.2:14     209.165.200.1:14   209.165.200.1:1029
icmp 209.165.200.226:1030192.168.1.2:15     209.165.200.1:15   209.165.200.1:1030
icmp 209.165.200.226:1031192.168.1.2:16     209.165.200.1:16   209.165.200.1:1031
icmp 209.165.200.226:1032192.168.1.2:17     209.165.200.1:17   209.165.200.1:1032
icmp 209.165.200.226:1033192.168.1.2:18     209.165.200.1:18   209.165.200.1:1033
icmp 209.165.200.226:1034192.168.1.2:19     209.165.200.1:19   209.165.200.1:1034
icmp 209.165.200.226:1035192.168.1.2:20     209.165.200.1:20   209.165.200.1:1035
icmp 209.165.200.226:1036192.168.1.2:21     209.165.200.1:21   209.165.200.1:1036
icmp 209.165.200.226:1037192.168.1.2:22     209.165.200.1:22   209.165.200.1:1037
icmp 209.165.200.226:1038192.168.1.2:23     209.165.200.1:23   209.165.200.1:1038
icmp 209.165.200.226:1039192.168.1.2:24     209.165.200.1:24   209.165.200.1:1039
icmp 209.165.200.226:10192.168.1.3:10     209.165.200.1:10   209.165.200.1:10
icmp 209.165.200.226:11192.168.1.3:11     209.165.200.1:11   209.165.200.1:11
icmp 209.165.200.226:12192.168.1.3:12     209.165.200.1:12   209.165.200.1:12
icmp 209.165.200.226:13192.168.1.3:13     209.165.200.1:13   209.165.200.1:13
icmp 209.165.200.226:14192.168.1.3:14     209.165.200.1:14   209.165.200.1:14
```

Обратите внимание, что внутренний глобальный адрес одинаков для обоих сеансов. 

#### Вопрос:
Как маршрутизатор отслеживает, куда идут ответы? 

По присвоенным уникальным номерам портов


d.	PAT в пул является очень эффективным решением для малых и средних организаций. Тем не менее есть неиспользуемые адреса IPv4, задействованные в этом сценарии. Мы перейдем к PAT с перегрузкой интерфейса, чтобы устранить эту трату IPv4 адресов. Остановите ping на PC-A и PC-B с помощью комбинации клавиш Control-C, затем очистите трансляции и статистику:

```
R1# clear ip nat translations * 
R1# clear ip nat statistics 
```

#### Шаг 4. На R1 удалите команды преобразования nat pool.

Опять же, наш список доступа (список доступа 1) по-прежнему корректен для сетевого сценария, поэтому нет необходимости воссоздавать его. Кроме того, внутренний и внешний интерфейсы не меняются. Чтобы начать работу с PAT к интерфейсу, очистите конфигурацию, удалив пул NAT и команду, связывающую ACL и пул вместе.

```
R1(config)# no ip nat inside source list 1 pool PUBLIC_ACCESS overload 
R1(config)# no ip nat pool PUBLIC_ACCESS
```

#### Шаг 5. Добавьте команду PAT overload, указав внешний интерфейс.
Добавьте команду `PAT`, которая вызовет перегрузку внешнего интерфейса.

`R1(config)# ip nat inside source list 1 interface g0/0/0 overload `

#### Шаг 6. Протестируйте и проверьте конфигурацию. 

a.	Давайте проверим PAT, чтобы интерфейс работал. С PC-B,  запустите эхо-запрос интерфейса Lo1 (209.165.200.1) на R2. Если эхо-запрос не прошел, выполните отладку. На R1 отобразите таблицу NAT на R1 с помощью команды `show ip nat translations`.

```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.230:207192.168.1.3:207    209.165.200.1:207  209.165.200.1:207
icmp 209.165.200.230:208192.168.1.3:208    209.165.200.1:208  209.165.200.1:208
icmp 209.165.200.230:209192.168.1.3:209    209.165.200.1:209  209.165.200.1:209
icmp 209.165.200.230:210192.168.1.3:210    209.165.200.1:210  209.165.200.1:210
```

b.	Сделайте трафик с нескольких устройств для наблюдения PAT. На PC-A и PC-B используйте параметр `-t` с командой ping для отправки безостановочного ping на интерфейс Lo1 R2 (ping -t 209.165.200.1). На S1 и S2 выполните привилегированную команду `exec ping 209.165.200.1` повторить 2000. Затем вернитесь к R1 и выполните команду `show ip nat translations`.

```
R1# show ip nat translations
Pro Inside global Inside local Outside local Outside global
icmp 209.165.200.230:728192.168.1.3:728    209.165.200.1:728  209.165.200.1:728
icmp 209.165.200.230:729192.168.1.3:729    209.165.200.1:729  209.165.200.1:729
icmp 209.165.200.230:72192.168.1.12:72    209.165.200.1:72   209.165.200.1:72
icmp 209.165.200.230:730192.168.1.3:730    209.165.200.1:730  209.165.200.1:730
icmp 209.165.200.230:731192.168.1.3:731    209.165.200.1:731  209.165.200.1:731
icmp 209.165.200.230:732192.168.1.3:732    209.165.200.1:732  209.165.200.1:732
icmp 209.165.200.230:733192.168.1.3:733    209.165.200.1:733  209.165.200.1:733
icmp 209.165.200.230:734192.168.1.3:734    209.165.200.1:734  209.165.200.1:734
icmp 209.165.200.230:735192.168.1.3:735    209.165.200.1:735  209.165.200.1:735
icmp 209.165.200.230:736192.168.1.3:736    209.165.200.1:736  209.165.200.1:736
icmp 209.165.200.230:737192.168.1.3:737    209.165.200.1:737  209.165.200.1:737
icmp 209.165.200.230:738192.168.1.3:738    209.165.200.1:738  209.165.200.1:738
icmp 209.165.200.230:739192.168.1.3:739    209.165.200.1:739  209.165.200.1:739
icmp 209.165.200.230:73192.168.1.12:73    209.165.200.1:73   209.165.200.1:73
icmp 209.165.200.230:740192.168.1.3:740    209.165.200.1:740  209.165.200.1:740
icmp 209.165.200.230:741192.168.1.3:741    209.165.200.1:741  209.165.200.1:741
icmp 209.165.200.230:74192.168.1.12:74    209.165.200.1:74   209.165.200.1:74
```

Теперь все внутренние глобальные адреса сопоставляются с IP-адресом интерфейса g0/0/0.
Остановите все пинги на PC-A и PC-B, используя комбинацию клавиш CTRL-C.

### Часть 4. Настройка и проверка статического NAT для IPv4.

В части 4 будет настроена статическая NAT таким образом, чтобы PC-A был доступен напрямую из Интернета. PC-A будет доступен из R2 по адресу 209.165.200.229.

**Примечание:** Конфигурация, которую вы собираетесь завершить, не соответствует рекомендуемым практикам для шлюзов, подключенных к Интернету. Эта лаборатория полностью опускает стандартные методы безопасности, чтобы сосредоточиться на успешной конфигурации статического NAT. В производственной среде решающее значение для удовлетворения этого требования будет иметь тщательная координация между сетевой инфраструктурой и группами безопасности. 

#### Шаг 1. На R1 очистите текущие трансляции и статистику.

Откройте окно конфигурации

```
R1# clear ip nat translations * 
R1# clear ip nat statistics 
```

#### Шаг 2. На R1 настройте команду NAT, необходимую для статического сопоставления внутреннего адреса с внешним адресом.

Для этого шага настройте статическое сопоставление между 192.168.1.11 и 209.165.200.1 с помощью следующей команды:

`R1(config)# ip nat inside source static 192.168.1.2 209.165.200.229 `

#### Шаг 3. Протестируйте и проверьте конфигурацию.

a.	Давайте проверим, что статический NAT работает. На R1 отобразите таблицу NAT на R1 с помощью команды show ip nat translations, и вы увидите статическое сопоставление.

```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
---  209.165.200.229   192.168.1.2        ---                ---
```

b.	Таблица перевода показывает, что статическое преобразование действует. Проверьте это, запустив ping  с R2 на 209.165.200.229. Пинги должны работать.
Примечание. Возможно, вам придется отключить брандмауэр ПК для работы pings.

```
R2#ping 209.165.200.229

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 209.165.200.229, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms
```

c.	На R1 отобразите таблицу NAT на R1 с помощью команды show ip nat translations, и вы увидите статическое сопоставление и преобразование на уровне порта для входящих pings.

```
R1#show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 209.165.200.229:2 192.168.1.2:2      209.165.200.225:2  209.165.200.225:2
icmp 209.165.200.229:3 192.168.1.2:3      209.165.200.225:3  209.165.200.225:3
icmp 209.165.200.229:4 192.168.1.2:4      209.165.200.225:4  209.165.200.225:4
icmp 209.165.200.229:5 192.168.1.2:5      209.165.200.225:5  209.165.200.225:5
---  209.165.200.229   192.168.1.2        ---                ---
```

Это подтверждает, что статический NAT работает.
