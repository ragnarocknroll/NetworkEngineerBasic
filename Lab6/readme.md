## Лабораторная работа - Внедрение маршрутизации между виртуальными локальными сетями 

### Топология

![](/Lab6/Images/00_topology.jpg)

### Таблица адресации

Устройство | Интерфейс   | IP-адрес      | Маска подсети | Шлюз по умолчанию |
-----------|-------------|---------------|---------------|-------------------|
R1         | G0/0/1.10   | 192.168.10.1  | 255.255.255.0 | -                 |
|          | G0/0/1.20   | 192.168.20.1  | 255.255.255.0 | -                 |
|          | G0/0/1.30   | 192.168.30.1  | 255.255.255.0 | -                 |
|          | G0/0/1.1000 | -             | -             | -                 |
S1         | VLAN 10     | 192.168.10.11 | 255.255.255.0 | 192.168.10.1      |
S2         | VLAN 10     | 192.168.10.12 | 255.255.255.0 | 192.168.10.1      |
PC-A       | NIC         | 192.168.20.3  | 255.255.255.0 | 192.168.20.1      |
PC-B       | NIC         | 192.168.30.3  | 255.255.255.0 | 192.168.30.1      |

Таблица VLAN

VLAN | Имя         | Назначенный нтерфейс         |
-----|-------------|------------------------------|
10   | Управление  | S1: VLAN 10                  |
|    |             | S2: VLAN 10                  |
20   |	Sales	   | S1: F0/6                     |
30	 | Operations  | S2: F0/18                    |
999	 | Parking_Lot |С1: F0/2-4, F0/7-24, G0/1-2   |
|    |             |С2: F0/2-17, F0/19-24, G0/1-2 |
1000 | Собственная | —                            |

### Задачи

#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Создание сетей VLAN и назначение портов коммутатора
#### Часть 3. Настройка транка 802.1Q между коммутаторами.
#### Часть 4. Настройка маршрутизации между сетями VLAN
#### Часть 5. Проверка, что маршрутизация между VLAN работает

### Общие сведения/сценарий

В целях повышения производительности сети большие широковещательные домены 2-го уровня делят на домены меньшего размера. Для этого современные коммутаторы используют виртуальные локальные сети (VLAN). VLAN также можно использовать в качестве меры безопасности, отделяя конфиденциальный трафик данных от остальной части сети. Сети VLAN облегчают процесс проектирования сети, обеспечивающей помощь в достижении целей организации. Для связи между VLAN требуется устройство, работающее на уровне 3 модели OSI. Добавление маршрутизации между VLAN позволяет организации разделять и разделять широковещательные домены, одновременно позволяя им обмениваться данными друг с другом.

Транковые каналы сети VLAN используются для распространения сетей VLAN по различным устройствам. Транковые каналы разрешают передачу трафика из множества сетей VLAN через один канал, не нанося вред идентификации и сегментации сети VLAN. Особый вид маршрутизации между VLAN, называемый «Router-on-a-Stick», использует магистраль от маршрутизатора к коммутатору, чтобы все VLAN могли переходить к маршрутизатору.

В этой лабораторной работе необходимо создать VLAN на обоих коммутаторах в топологии, назначить VLAN для коммутации портов доступа, убедиться, что VLAN работают должным образом, создадать транки VLAN между двумя коммутаторами и между S1 и R1, и настройте маршрутизацию между VLAN на R1 для разрешения связи между хостами в разных VLAN независимо от подсети, в которой находится хост.

**Примечание:** Маршрутизаторы, используемые в практических лабораторных работах CCNA, - это Cisco 4221 с Cisco IOS XE Release 16.9.4 (образ universalk9). В лабораторных работах используются коммутаторы Cisco Catalyst 2960 с Cisco IOS версии 15.2(2) (образ lanbasek9). Можно использовать другие маршрутизаторы, коммутаторы и версии Cisco IOS. В зависимости от модели устройства и версии Cisco IOS доступные команды и результаты их выполнения могут отличаться от тех, которые показаны в лабораторных работах. Правильные идентификаторы интерфейса см. в сводной таблице по интерфейсам маршрутизаторов в конце лабораторной работы.

**Примечание:** Необходимо убедиться, что у всех маршрутизаторов и коммутаторов была удалена начальная конфигурация.

#### Необходимые ресурсы

+ 1 Маршрутизатор (Cisco 4221 с универсальным образом Cisco IOS XE версии 16.9.4 или аналогичным)
+ 2 коммутатора (Cisco 2960 с операционной системой Cisco IOS 15.2(2) (образ lanbasek9) или аналогичная модель)
+ 2 ПК (ОС Windows с программой эмуляции терминалов, такой как Tera Term)
+ Консольные кабели для настройки устройств Cisco IOS через консольные порты.
+ Кабели Ethernet, расположенные в соответствии с топологией

### Инструкции
### Часть 1. Создание сети и настройка основных параметров устройства

В первой части лабораторной работы вам предстоит создать топологию сети и настроить базовые параметры для узлов ПК и коммутаторов.

#### Шаг 1. Создайте сеть согласно топологии.

Подключите устройства, как показано в топологии, и подсоедините необходимые кабели.

#### Шаг 2. Настройте базовые параметры для маршрутизатора.

a.	Необходимо подключиться к маршрутизатору с помощью консоли и активируйте привилегированный режим EXEC.

b.	Необходимо войти в режим конфигурации.

c.	Необходимо назначить маршрутизатору имя устройства.

d.	Необходимо отключить поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

e.	Необходимо назначить class в качестве зашифрованного пароля привилегированного режима EXEC.

f.	Необходимо назначить cisco в качестве пароля консоли и включите вход в систему по паролю.

g.	Необходимо установить cisco в качестве пароля виртуального терминала и активируйте вход.

h.	Необходимо зашифровать открытые пароли.

i.	Необходимо создать баннер с предупреждением о запрете несанкционированного доступа к устройству.

j.	Необходимо сохранить текущую конфигурацию в файл загрузочной конфигурации.

k.	Необходимо настроить на маршрутизаторе время.

Результат настройки R1:
```
R1#show startup-config
Using 775 bytes
!
version 15.4
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
 no ip address
 duplex auto
 speed auto
 shutdown
!
interface GigabitEthernet0/0/1
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
ATTENTION!!! Unauthorized acces is strictly prohibited! ^C
!
!
!
!
!
line con 0
 password 7 0822455D0A16
 logging synchronous
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
R1#
R1#
R1#
R1#show clock
21:38:13.341 UTC Tue Nov 7 2023
```

#### Шаг 3. Необходимо настроить базовые параметры каждого коммутатора.

a.	Необходимо присвоить коммутатору имя устройства.

b.	Необходимо отключить поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

c.	Необходимо назначить class в качестве зашифрованного пароля привилегированного режима EXEC.

d.	Необходимо назначить cisco в качестве пароля консоли и включите вход в систему по паролю.

e.	Необходимо установить cisco в качестве пароля виртуального терминала и активируйте вход.

f.	Необходимо зашифровать открытые пароли.

g.	Необходимо создать баннер с предупреждением о запрете несанкционированного доступа к устройству.

h.	Необходимо настроить на коммутаторах время.

i.	Необходимо сохранить текущей конфигурации в качестве начальной.

Результат настройки S1:
```
S1#show clock
22:10:54.168 UTC Thu Nov 7 2024
S1#
S1#
S1#show start
S1#show startup-config 
Using 1317 bytes
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
!
interface FastEthernet0/3
!
interface FastEthernet0/4
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
!
interface FastEthernet0/8
!
interface FastEthernet0/9
!
interface FastEthernet0/10
!
interface FastEthernet0/11
!
interface FastEthernet0/12
!
interface FastEthernet0/13
!
interface FastEthernet0/14
!
interface FastEthernet0/15
!
interface FastEthernet0/16
!
interface FastEthernet0/17
!
interface FastEthernet0/18
!
interface FastEthernet0/19
!
interface FastEthernet0/20
!
interface FastEthernet0/21
!
interface FastEthernet0/22
!
interface FastEthernet0/23
!
interface FastEthernet0/24
!
interface GigabitEthernet0/1
!
interface GigabitEthernet0/2
!
interface Vlan1
 no ip address
 shutdown
!
banner motd ^C
ATTENTION!!! Unauthorized acces is strictly prohibited! ^C
!
!
!
line con 0
 password 7 0822455D0A16
 logging synchronous
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

Результат настройки S2:
```
S2#show clock
22:19:9.391 UTC Thu Nov 7 2024
S2#show start
S2#show startup-config 
Using 1317 bytes
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
!
interface FastEthernet0/3
!
interface FastEthernet0/4
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
!
interface FastEthernet0/8
!
interface FastEthernet0/9
!
interface FastEthernet0/10
!
interface FastEthernet0/11
!
interface FastEthernet0/12
!
interface FastEthernet0/13
!
interface FastEthernet0/14
!
interface FastEthernet0/15
!
interface FastEthernet0/16
!
interface FastEthernet0/17
!
interface FastEthernet0/18
!
interface FastEthernet0/19
!
interface FastEthernet0/20
!
interface FastEthernet0/21
!
interface FastEthernet0/22
!
interface FastEthernet0/23
!
interface FastEthernet0/24
!
interface GigabitEthernet0/1
!
interface GigabitEthernet0/2
!
interface Vlan1
 no ip address
 shutdown
!
banner motd ^C
ATTENTION!!! Unauthorized acces is strictly prohibited! ^C
!
!
!
line con 0
 password 7 0822455D0A16
 logging synchronous
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

#### Шаг 4. Необходимо настроить узлы ПК.

Адреса ПК можно посмотреть в таблице адресации.

Результат настройки PC-A:
```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::203:E4FF:FE43:C1DE
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.20.3
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.20.1

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0
```

Результат настройки PC-B:
```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::200:CFF:FE71:253E
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.30.3
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.30.1

Bluetooth Connection:

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: ::
   IPv6 Address....................: ::
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: ::
                                     0.0.0.0

C:\>
```

### Часть 2. Создание сетей VLAN и назначение портов коммутатора

Во второй части необходимо создать VLAN, как указано в таблице выше, на обоих коммутаторах. Затем вы назначите VLAN соответствующему интерфейсу и проверите настройки конфигурации. Выполните следующие задачи на каждом коммутаторе.

#### Шаг 1. Необходимо создать сети VLAN на коммутаторах.

a.	Необходимо создать и назвать необходимые VLAN на каждом коммутаторе из таблицы выше.

Процедура выполняется из режима конфигурации с помощью команды `vlan *номер*`, после обращения к vlan'у можно задать его имя с помощью команды `name *имя*`

Пример:
```
S2(config-vlan)#vlan 999
S2(config-vlan)#
S2(config-vlan)#
S2(config-vlan)#name Parking_Lot
```

Привожу результат для S1:
```
S1#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
10   Management                       active    
20   Sales                            active    
30   Operations                       active    
999  Parking_Lot                      active    
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
20   enet  100020     1500  -      -      -        -    -        0      0
30   enet  100030     1500  -      -      -        -    -        0      0
999  enet  100999     1500  -      -      -        -    -        0      0
1000 enet  101000     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0   
1003 tr    101003     1500  -      -      -        -    -        0      0   
1004 fdnet 101004     1500  -      -      -        ieee -        0      0   
1005 trnet 101005     1500  -      -      -        ibm  -        0      0   

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------

Remote SPAN VLANs
------------------------------------------------------------------------------

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
```

Теперь привожу результат для S2:
```
S2#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
10   Management                       active    
20   Sales                            active    
30   Operations                       active    
999  Parking_Lot                      active    
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
20   enet  100020     1500  -      -      -        -    -        0      0
30   enet  100030     1500  -      -      -        -    -        0      0
999  enet  100999     1500  -      -      -        -    -        0      0
1000 enet  101000     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0   
1003 tr    101003     1500  -      -      -        -    -        0      0   
1004 fdnet 101004     1500  -      -      -        ieee -        0      0   
1005 trnet 101005     1500  -      -      -        ibm  -        0      0   

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------

Remote SPAN VLANs
------------------------------------------------------------------------------

Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
```

b.	Необходимо настроить интерфейс управления и шлюз по умолчанию на каждом коммутаторе, используя информацию об IP-адресе в таблице адресации. 

Выполняю с помощью команд:
```
S1(config)# interface vlan 10
S1(config-if)# ip address 192.168.10.11 255.255.255.0
S1(config-if)# no shutdown
S1(config-if)# exit
S1(config)# ip default-gateway 192.168.10.1
```

Привожу результат для S1 по команде show running-config:
```
!
interface Vlan10
 ip address 192.168.10.11 255.255.255.0
!
ip default-gateway 192.168.10.1
!
```

Аналогично для S2:
```
!
interface Vlan10
 ip address 192.168.10.12 255.255.255.0
!
ip default-gateway 192.168.10.1
!
```

c.	Необходимо назначить все неиспользуемые порты коммутатора VLAN Parking_Lot, настроить их для статического режима доступа и административно деактивируйте их.

Выполняю для S1 с помощью последовательности команд:
```
S1(config)#
S1(config)#in
S1(config)#interface  ra
S1(config)#interface  range f0/2 - 4, f0/7 - 24, G0/1 - 2
S1(config-if-range)#swi
S1(config-if-range)#switchport 
S1(config-if-range)#switchport mod
S1(config-if-range)#switchport mode acc
S1(config-if-range)#switchport mode access
S1(config-if-range)#
S1(config-if-range)#
S1(config-if-range)#switchport ac
S1(config-if-range)#switchport access 
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#
S1(config-if-range)#
S1(config-if-range)#shutdown
```
Для S2:
```
S2(config)#in
S2(config)#interface ra
S2(config)#interface range f0/2 - 17, f0/19 - 24, g0/1 - 2
S2(config-if-range)#sw
S2(config-if-range)#switchport mo
S2(config-if-range)#switchport mode ac
S2(config-if-range)#switchport mode access
S2(config-if-range)#switch
S2(config-if-range)#switchport
S2(config-if-range)#switchport acc
S2(config-if-range)#switchport access vlan 999
S2(config-if-range)#shutdown
```


**Примечание:** Команда `interface range` полезна для выполнения этой задачи с минимальным количеством команд.

#### Шаг 2. Необходимо назначить сети VLAN соответствующим интерфейсам коммутатора.

a.	Необходимо назначить используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.

Выполняю с помощью последовательности команд для S1:
```
S1(config)#interface f0/6
S1(config-if)#swi
S1(config-if)#switchport mode acc
S1(config-if)#switchport mode access 
S1(config-if)#
S1(config-if)#switchport access vlan 20 
```

Для S2:
```
S2(config)#interface f0/18
S2(config-if)#si
S2(config-if)#sw
S2(config-if)#switchport mo
S2(config-if)#switchport mode acces
S2(config-if)#
S2(config-if)#
S2(config-if)#
S2(config-if)#sw
S2(config-if)#switchport ac
S2(config-if)#switchport access vlan 30
```

b.	Необходимо убедиться, что VLAN назначены на правильные интерфейсы.

Привожу результат для S1:
```
S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/5
10   Management                       active    
20   Sales                            active    Fa0/6
30   Operations                       active    
999  Parking_Lot                      active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
S1# 
```

Для S2:
```
S2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1
10   Management                       active    
20   Sales                            active    
30   Operations                       active    Fa0/18
999  Parking_Lot                      active    Fa0/2, Fa0/3, Fa0/4, Fa0/5
                                                Fa0/6, Fa0/7, Fa0/8, Fa0/9
                                                Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15, Fa0/16, Fa0/17
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active
```

### Часть 3. Конфигурация магистрального канала стандарта 802.1Q между коммутаторами

В части 3 необходимо вручную настроить интерфейс F0/1 как транк.

#### Шаг 1. Ручная настройка магистрального интерфейса F0/1 на коммутаторах S1 и S2.

a.	Необходима настройка статического транкинга на интерфейсе F0/1 для обоих коммутаторов.

Задаю последовательность команд для S1:
```
S1(config)# interface f0/1
S1(config-if)# switchport mode trunk
```
Аналогично для S2:
```
S2(config)# interface f0/1
S2(config-if)# switchport mode trunk
```

b.	Необходимо установить native VLAN 1000 на обоих коммутаторах.

Для обоих коммутаторов выполняю с помощью команды `switchport trunk native vlan 1000`

c.	Необходимо указать, что VLAN 10, 20, 30 и 1000 могут проходить по транку.

Для обоих коммутаторов выполняю с помощью команды1 `switchport trunk allowed vlan 10,20,30,1000`

d.	Необходимо проверить транки, native VLAN и разрешенные VLAN через транк.

Привожу результат проверки для S1:
```
S1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,30,1000

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       none

S1#
```

Для S2:
```
S2#show interface trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/1       10,20,30,1000

Port        Vlans allowed and active in management domain
Fa0/1       10,20,30,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       10,20,30,1000
```


#### Шаг 2. Ручная настройка магистрального интерфейса F0/5 на коммутаторе S1.

a.	Необходимо настроить интерфейс S1 F0/5 с теми же параметрами транка, что и F0/1. Это транк до маршрутизатора.

```
S1(config)# interface f0/5
S1(config-if)# switchport mode trunk
S1(config-if)# switchport trunk native vlan 1000
S1(config-if)# switchport trunk allowed vlan 10,20,30,1000
```

b.	Необходимо сохранить текущую конфигурацию в файл загрузочной конфигурации.

Выполняю для каждого коммутатора копирование текущих настроек в загрузочные.

c.	Проверка транкинга.

Порт F0/5 отображается отключенным, причина раскрыта в ответе на вопрос.


**Вопрос:**
Что произойдет, если G0/0/1 на R1 будет отключен?

**Ответ:**
Интерфейс F0/5 коммутатора S1 будет не активен, что видно при проверке с помощью команды `show interface f0/5`

### Часть 4. Настройка маршрутизации между сетями VLAN

#### Шаг 1. Настройка маршрутизатора.

a.	При необходимости можно активировать интерфейс G0/0/1 на маршрутизаторе.

Выполняю с помощью последовательности команд:
```
R1(config)#interface g0/0/1
R1(config-if)#no sh
R1(config-if)#no shutdown 
```

b.	Необходимо настроить подинтерфейсы для каждого VLAN, как указано в таблице IP-адресации. Все подинтерфейсы используют инкапсуляцию 802.1Q. Необходимо убедиться, что подинтерфейсу для native VLAN не назначен IP-адрес. Необходимо включить описание для каждого подинтерфейса.

Выполняю с помощью последовательности команд:
```
R1(config)# interface g0/0/1.10
R1(config-subif)# encapsulation dot1q 10
R1(config-subif)# description Management Network
R1(config-subif)# ip address 192.168.10.1 255.255.255.0

R1(config-subif)# interface g0/0/1.20
R1(config-subif)# encapsulation dot1q 20
R1(config-subif)# description Sales Network
R1(config-subif)# ip address 192.168.20.1 255.255.255.0

R1(config-subif)# interface g0/0/1.30
R1(config-subif)# encapsulation dot1q 30
R1(config-subif)# description Operations Network
R1(config-subif)# ip address 192.168.30.1 255.255.255.0

R1(config-subif)# interface g0/0/1.1000
R1(config-subif)# encapsulation dot1q 1000 native
R1(config-subif)# description Native VLAN
```
c.	Необходимо убедиться, что вспомогательные интерфейсы работают

Привожу результат проверки:
```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.10192.168.10.1    YES manual up                    up 
GigabitEthernet0/0/1.20192.168.20.1    YES manual up                    up 
GigabitEthernet0/0/1.30192.168.30.1    YES manual up                    up 
GigabitEthernet0/0/1.1000unassigned      YES unset  up                    up 
Vlan1                  unassigned      YES unset  administratively down down
R1#
```

### Часть 5. Проверка работы маршрутизации между VLAN

#### Шаг 1. Необходимо выполнить тесты с PC-A.

**Примечание:** Возможно, придется отключить брандмауэр ПК для работы ping

a.	Необходимо отправить эхо-запрос с PC-A на шлюз по умолчанию.

```
C:\>ping 192.168.20.1

Pinging 192.168.20.1 with 32 bytes of data:

Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255
Reply from 192.168.20.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.20.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

b.	Необходимо отправить эхо-запрос с PC-A на PC-B.

```
C:\>ping 192.168.30.3

Pinging 192.168.30.3 with 32 bytes of data:

Request timed out.
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127
Reply from 192.168.30.3: bytes=32 time<1ms TTL=127

Ping statistics for 192.168.30.3:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

c.	Необходимо отправить команду ping с компьютера PC-A на коммутатор S2.

```
C:\>ping 192.168.10.12

Pinging 192.168.10.12 with 32 bytes of data:

Request timed out.
Request timed out.
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254
Reply from 192.168.10.12: bytes=32 time<1ms TTL=254

Ping statistics for 192.168.10.12:
    Packets: Sent = 4, Received = 2, Lost = 2 (50% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0m
```

#### Шаг 2. Необходимо выполнить тесты с PC-B

В окне командной строки на PC-B необходимо выполнить команду tracert на адрес PC-A.

```
C:\>tracert 192.168.20.3

Tracing route to 192.168.20.3 over a maximum of 30 hops: 

  1   1 ms      0 ms      0 ms      192.168.30.1
  2   0 ms      0 ms      0 ms      192.168.20.3

Trace complete.
```
**Вопрос:**

Какие промежуточные IP-адреса отображаются в результатах?

**Ответ:**

Промежуточный адрес, который отобразился при выполнении этой команды, является адресом G0/0/1.30 на маршрутизаторе R1, он же является шлюзом для PC-B