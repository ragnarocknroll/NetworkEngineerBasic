## Лабораторная работа №4. Настройка IPv6-адресов на сетевых устройствах

### Топология

![](/Lab4/Images/00_topology.jpg)

### Таблица адресации

Устройство | Интерфейс | IPv6-адрес         | Link local IPv6-адрес | Длина префикса | Шлюз по умолчанию |
-----------|-----------|--------------------|-----------------------|----------------|-------------------|
R1         | G0/0/0    | 2001:db8:acad:a::1 | fe80::1               | 64             | —                 |
R1         | G0/0/1    | 2001:db8:acad:1::1 | fe80::1               | 64             | —                 |
S1         | VLAN 1    | 2001:db8:acad:1::b | fe80::b               | 64             | —                 |
PC-A       | NIC       | 2001:db8:acad:1::3 | SLAAC                 | 64             | fe80::1           |
PC-B       | NIC       | 2001:db8:acad:a::3 | SLAAC                 | 64             | fe80::1           |

### Задачи

#### Часть 1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора
#### Часть 2. Ручная настройка IPv6-адресов
#### Часть 3. Проверка сквозного соединения

### Общие сведения/сценарий

В этой лабораторной работе необходимо настроить хосты и интерфейсы устройств с IPv6-адресами.  Для просмотра индивидуальных и групповых IPv6-адресов необходимо использовать команду `show`. Также необходимо проверить сквозное соединение с помощью команд `ping` и `traceroute`.

**Примечание:** Маршрутизаторы, используемые в практических лабораторных работах CCNA, - это Cisco 4221 с Cisco IOS XE Release 16.9.4 (образ universalk9). В лабораторных работах используются коммутаторы Cisco Catalyst 2960 с Cisco IOS версии 15.2(2) (образ lanbasek9). Можно использовать другие маршрутизаторы, коммутаторы и версии Cisco IOS. В зависимости от модели устройства и версии Cisco IOS доступные команды и результаты их выполнения могут отличаться от тех, которые показаны в лабораторных работах. Правильные идентификаторы интерфейса см. в сводной таблице по интерфейсам маршрутизаторов в конце лабораторной работы.

**Примечание:** Убедитесь, что у всех маршрутизаторов и коммутаторов была удалена начальная конфигурация. Если вы не уверены, обратитесь к инструктору.

**Примечание:** Шаблон по умолчанию менеджера базы данных 2960 Switch Database Manager (SDM) не поддерживает IPv6. Перед назначением IPv6-адреса SVI VLAN 1 может понадобиться выполнение команды `sdm prefer dual-ipv4-and-ipv6 default` для включения **IPv6-адресации**.

**Примечание:** **Шаблон default bias**, который по умолчанию используется диспетчером SDM (диспетчер базы данных коммутатора), не предоставляет возможностей адресации IPv6. Необходимо убедиться, что SDM использует шаблон dual-ipv4-and-ipv6 или **lanbase-routing**. Новый шаблон будет использоваться после перезагрузки.

`S1# show sdm prefer`

Чтобы установить шаблон dual-ipv4-and-ipv6 в качестве шаблона SDM по умолчанию, выполните следующие действия:

```
S1# configure terminal
S1(config)# sdm prefer dual-ipv4-and-ipv6 default
S1(config)# end
S1# reload
```

### Необходимые ресурсы
+ 1 Маршрутизатор (Cisco 4221 с универсальным образом Cisco IOS XE версии 16.9.4 или аналогичным)
+ 1 коммутатор (Cisco 2960 с ПО Cisco IOS версии 15.2(2) с образом lanbasek9 или аналогичная модель)
+ 2 ПК (Windows и программа эмуляции терминала, такая как Tera Term)
+ Консольные кабели для настройки устройств Cisco IOS через консольные порты.
+ Кабели Ethernet, расположенные в соответствии с топологией

**Примечание:** Интерфейсы Gigabit Ethernet на маршрутизаторах Cisco 4221 определяют скорость автоматически, поэтому для подключения маршрутизатора к PC-B можно использовать прямой кабель Ethernet. При использовании другой модели маршрутизатора Cisco может возникнуть необходимость использовать перекрестный кабель Ethernet

### Инструкции

### Часть 1. Настройка топологии и конфигурация основных параметров маршрутизатора и коммутатора

После подключения сети, инициализации и перезагрузки маршрутизатора и коммутатора необходимо выполнить следующие действия:

#### Шаг 1. Настроить маршрутизатор.
Назначить имя хоста и настройте основные параметры устройства.

```
R1#show running-config 
Building configuration...

Current configuration : 1051 bytes
!
version 15.1
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
ipv6 unicast-routing
!
no ipv6 cef
!
!
!
!
license udi pid CISCO2911/K9 sn FTX1524OLBS-
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
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:A::1/64
 ipv6 enable
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
 ipv6 enable
!
interface GigabitEthernet0/2
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
 password 7 0822455D0A16
 logging synchronous
 login
!
line vty 0 4
 password 7 0822455D0A16
 login
!
!
!
end


R1#
```

#### Шаг 2. Настроить коммутатор.
Назначить имя хоста и настройте основные параметры устройства.

```
S1#show running-config 
Building configuration...

Current configuration : 1393 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname S1
!
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
 ipv6 address FE80::B link-local
 ipv6 address 2001:DB8:ACAD:1::B/64
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


S1#
```

### Часть 2. Ручная настройка IPv6-адресов

#### Шаг 1. Назначить IPv6-адреса интерфейсам Ethernet на R1.

a.	Назначить глобальные индивидуальные IPv6-адреса, указанные в таблице адресации обоим интерфейсам Ethernet на R1.
Открыть окно конфигурации

```
interface GigabitEthernet0/0
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:A::1/64
 ipv6 enable
!
interface GigabitEthernet0/1
 no ip address
 duplex auto
 speed auto
 ipv6 address FE80::1 link-local
 ipv6 address 2001:DB8:ACAD:1::1/64
 ipv6 enable
```

b.	Ввести команду `show ipv6 interface brief`, чтобы проверить, назначен ли каждому интерфейсу корректный индивидуальный IPv6-адрес.

```
R1#show ipv6 interface brief
GigabitEthernet0/0         [up/up]
    FE80::1
    2001:DB8:ACAD:A::1
GigabitEthernet0/1         [up/up]
    FE80::1
    2001:DB8:ACAD:1::1
GigabitEthernet0/2         [administratively down/down]
    unassigned
Vlan1                      [administratively down/down]
    unassigned
```

**Примечание.** Отображаемый локальный адрес канала основан на адресации **EUI-64**, которая автоматически использует MAC-адрес интерфейса для создания 128-битного локального IPv6-адреса канала.

c.	Чтобы обеспечить соответствие локальных адресов канала индивидуальному адресу, необходимо вручную ввести локальные адреса канала на каждом интерфейсе Ethernet на R1.

**Примечание.** Каждый интерфейс маршрутизатора относится к отдельной сети. Пакеты с локальным адресом канала никогда не выходят за пределы локальной сети, а значит, для обоих интерфейсов можно указывать один и тот же локальный адрес канала.

d.	Необходимо использовать выбранную команду, чтобы убедиться, что локальный адрес связи изменен на fe80::1.  

Закрыть окно конфигурации.

**Вопрос:**
Какие группы многоадресной рассылки назначены интерфейсу G0/0?

**Ответ:** 

```
R1#show ipv6 interface g0/0
GigabitEthernet0/0 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::1
  No Virtual link-local address(es):
  Global unicast address(es):
    2001:DB8:ACAD:A::1, subnet is 2001:DB8:ACAD:A::/64
  Joined group address(es):
    FF02::1
    FF02::2
    FF02::1:FF00:1
```
Группы многоадресной рассылки всех узлов FF02::1 и FF02::2, а также, группа многоадресной рассылки запрошенных узлов (FF02::1:FF00:1).


#### Шаг 2. Активируйте IPv6-маршрутизацию на R1.

a.	В командной строке на PC-B необходимо ввести команду `ipconfig`, чтобы получить данные IPv6-адреса, назначенного интерфейсу ПК.

**Вопрос:**
Назначен ли индивидуальный IPv6-адрес сетевой интерфейсной карте (NIC) на PC-B?

**Ответ:** 
Нет, не получен

b.	Необходимо активировать IPv6-маршрутизацию на R1 с помощью команды `IPv6 unicast-routing`.

Группа многоадресной рассылки всех узлов FF02::2, приведенная в листинге выше была получена после введения команды `IPv6 unicast-routing`


**Примечание.** Это позволит компьютерам получать IP-адреса и данные шлюза по умолчанию с помощью функции SLAAC (Stateless Address Autoconfiguration  - Автоконфигурация без сохранения состояния адреса).

c.	Теперь, когда R1 входит в группу многоадресной рассылки всех маршрутизаторов, необходимо еще раз ввести команду `ipconfig` на PC-B. Проверьте данные IPv6-адреса.

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::20C:CFFF:FE42:8E4B   
   IPv6 Address....................: 2001:DB8:ACAD:A:20C:CFFF:FE42:8E4B
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

**Вопрос:**
Почему PC-B получил глобальный префикс маршрутизации и идентификатор подсети, которые вы настроили на R1?

**Ответ:** 
Маршрутизатор R1 G0/0 теперь входит в группу многоадресной рассылки FF02::2. Это позволяет ему отправлять сообщения Router Advertisement с информацией об адресе глобальной сети и идентификаторе подсети на все узлы локальной сети. R1 отправил адрес локальной сети link-local FE80::1 в качестве шлюза по умолчанию (Deaful gateway). В результате компьютеры получают свой IP-адрес и шлюз по умолчанию через SLAAC

#### Шаг 4. Назначьте компьютерам статические IPv6-адреса.
a.	Необходимо открыть окно **Свойства Ethernet** для каждого ПК и назначить адресацию IPv6.

P-A:

![](/Lab4/Images/01_conf-PC-A.jpg)

PC-B:

![](/Lab4/Images/02_conf-PC-B.jpg)


b.	Необходимо убедиться, что оба компьютера имеют правильную информацию адреса IPv6. Каждый компьютер должен иметь два глобальных адреса IPv6: один статический и один SLACC.

**Примечание.** При выполнении работы в среде Cisco Packet Tracer необходимо установаить статический и SLACC адреса на компьютеры последовательно, отразив результаты в отчете

Проверка адресов для PC-A:

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::203:E4FF:FE45:D3EA
   IPv6 Address....................: 2001:DB8:ACAD:1::3
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

Проверка адресов для PC-B:

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::20C:CFFF:FE42:8E4B
   IPv6 Address....................: 2001:DB8:ACAD:A::3
   IPv4 Address....................: 0.0.0.0
   Subnet Mask.....................: 0.0.0.0
   Default Gateway.................: FE80::1
                                     0.0.0.0
```

### Часть 3. Проверка сквозного подключения

С PC-A необходимо отправить эхо-запрос на FE80::1. Это локальный адрес канала, назначенный G0/1 на R1.

```
C:\>ping fe80::1

Pinging fe80::1 with 32 bytes of data:

Reply from FE80::1: bytes=32 time<1ms TTL=255
Reply from FE80::1: bytes=32 time<1ms TTL=255
Reply from FE80::1: bytes=32 time<1ms TTL=255
Reply from FE80::1: bytes=32 time<1ms TTL=255

Ping statistics for FE80::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
 ```

Необходимо отправить эхо-запрос на интерфейс управления S1 с PC-A.

```
C:\>ping fe80::b

Pinging fe80::b with 32 bytes of data:

Reply from FE80::B: bytes=32 time=1ms TTL=255
Reply from FE80::B: bytes=32 time<1ms TTL=255
Reply from FE80::B: bytes=32 time<1ms TTL=255
Reply from FE80::B: bytes=32 time<1ms TTL=255

Ping statistics for FE80::B:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

C:\>ping 2001:db8:acad:1::b

Pinging 2001:db8:acad:1::b with 32 bytes of data:

Reply from 2001:DB8:ACAD:1::B: bytes=32 time=1ms TTL=255
Reply from 2001:DB8:ACAD:1::B: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:1::B: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:1::B: bytes=32 time<1ms TTL=255

Ping statistics for 2001:DB8:ACAD:1::B:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

Необходимо ввести команду `tracert` на PC-A, чтобы проверить наличие сквозного подключения к PC-B.

```
C:\>tracert 2001:db8:acad:a::3

Tracing route to 2001:db8:acad:a::3 over a maximum of 30 hops: 

  1   0 ms      0 ms      0 ms      2001:DB8:ACAD:1::1
  2   0 ms      0 ms      0 ms      2001:DB8:ACAD:A::3

Trace complete.
```

С PC-B необходимо отправить эхо-запрос на PC-A.

```
C:\>ping 2001:db8:acad:1::3

Pinging 2001:db8:acad:1::3 with 32 bytes of data:

Reply from 2001:DB8:ACAD:1::3: bytes=32 time<1ms TTL=127
Reply from 2001:DB8:ACAD:1::3: bytes=32 time<1ms TTL=127
Reply from 2001:DB8:ACAD:1::3: bytes=32 time<1ms TTL=127
Reply from 2001:DB8:ACAD:1::3: bytes=32 time<1ms TTL=127

Ping statistics for 2001:DB8:ACAD:1::3:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

С PC-B необходимо отправить эхо-запрос на локальный адрес канала G0/0 на R1.

```
C:\>ping 2001:db8:acad:a::1

Pinging 2001:db8:acad:a::1 with 32 bytes of data:

Reply from 2001:DB8:ACAD:A::1: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:A::1: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:A::1: bytes=32 time<1ms TTL=255
Reply from 2001:DB8:ACAD:A::1: bytes=32 time<1ms TTL=255

Ping statistics for 2001:DB8:ACAD:A::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```
**Примечание.**  В случае отсутствия сквозного подключения необходимо проверить, правильно ли указаны IPv6-адреса на всех устройствах.

**Вопросы для повторения:**
1.	Почему обоим интерфейсам Ethernet на R1 можно назначить один и тот же локальный адрес канала — FE80::1?

Протокол IPv6 позволяет назначить глобальный и link-local адреса на один и тот же порт. При этом он позволяет использовать адреса link-local только внутри каждой локальной сети, потому что пакеты только внутри такой локальной сети

2.	Какой идентификатор подсети в индивидуальном IPv6-адресе 2001:db8:acad::aaaa:1234/64?

Четыре нуля, сокращенные думя двоеточиями, что допустимо для IPv6


### Сводная таблица по интерфейсам маршрутизаторов
| Модель маршрутизатора | Интерфейс Ethernet № 1          | Интерфейс Ethernet № 2          | Последовательный интерфейс № 1 | Последовательный интерфейс № 2|
|-----------------------|---------------------------------|---------------------------------|--------------------------------|-------------------------------|
| 1800                  | Fast Ethernet 0/0 (F0/0)        | Fast Ethernet 0/1 (F0/1)        | Serial 0/0/0 (S0/0/0)          | Serial 0/0/1 (S0/0/1)
| 1900                  | Gigabit Ethernet 0/0 (G0/0)     | Gigabit Ethernet 0/1 (G0/1)     | Serial 0/0/0 (S0/0/0)          | Serial 0/0/1 (S0/0/1)
| 2801                  | Fast Ethernet 0/0 (F0/0)        | Fast Ethernet 0/1 (F0/1)        | Serial 0/1/0 (S0/1/0)          | Serial 0/1/1 (S0/1/1)
| 2811                  | Fast Ethernet 0/0 (F0/0)        | Fast Ethernet 0/1 (F0/1)        | Serial 0/0/0 (S0/0/0)          | Serial 0/0/1 (S0/0/1)
| 2900                  | Gigabit Ethernet 0/0 (G0/0)     | Gigabit Ethernet 0/1 (G0/1)     | Serial 0/0/0 (S0/0/0)          | Serial 0/0/1 (S0/0/1)
| 4221                  | Gigabit Ethernet 0/0/0 (G0/0/0) | Gigabit Ethernet 0/0/1 (G0/0/1) | Serial 0/1/0 (S0/1/0)          | Serial 0/1/1 (S0/1/1)
| 4300                  | Gigabit Ethernet 0/0/0 (G0/0/0) | Gigabit Ethernet 0/0/1 (G0/0/1) | Serial 0/1/0 (S0/1/0)          | Serial 0/1/1 (S0/1/1)

**Примечание.**  Чтобы определить конфигурацию маршрутизатора, можно посмотреть на интерфейсы и установить тип маршрутизатора и количество его интерфейсов. Перечислить все комбинации конфигураций для каждого класса маршрутизаторов невозможно. Эта таблица содержит идентификаторы для возможных комбинаций интерфейсов Ethernet и последовательных интерфейсов на устройстве. Другие типы интерфейсов в таблице не представлены, хотя они могут присутствовать в данном конкретном маршрутизаторе. В качестве примера можно привести интерфейс ISDN BRI. Строка в скобках — это официальное сокращение, которое можно использовать в командах Cisco IOS для обозначения интерфейса.
