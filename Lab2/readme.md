## Лабораторная работа 2.  Просмотр таблицы MAC-адресов коммутатора 
### 	Топология

![](/Lab2/images/00-topology.jpg)

### Таблица адресации

Устройство | Интерфейс | Ip-адрес / префикс| Маска подсети |
-----------|-----------|-------------------|---------------|
S1         | VLAN 1    | 192.168.1.11/24   | 255.255.255.0 |
S2         | VLAN 1    | 192.168.1.12/24   | 255.255.255.0 |
PC-A       | NIC       | 192.168.1.1/24    | 255.255.255.0 |
PC-B       | NIC       | 192.168.1.2/24    | 255.255.255.0 |

### Цели

#### Часть 1. Создание и настройка сети**
#### Часть 2. Изучение таблицы МАС-адресов коммутатора**

#### Общие сведения/сценарий

Коммутатор локальной сети на уровне 2 предназначен для доставки кадров Ethernet всем узловым устройствам в локальной сети (LAN). Он записывает МАС-адреса узлов, отображаемые в сети, и сопоставляет их с собственными портами коммутатора Ethernet. Этот процесс называется созданием таблицы МАС-адресов. Получив кадр от ПК, коммутатор изучает МАС-адреса источника и назначения кадра. MAC-адрес источника регистрируется и сопоставляется с портом коммутатора, от которого он был получен. Затем по таблице MAC-адресов определяется МАС-адрес назначения. Если MAC-адрес назначения известен, кадр пересылается через соответствующий порт коммутатора, связанный с этим MAC-адресом. Если MAC-адрес неизвестен, то кадр отправляется по широковещательной рассылке через все порты коммутатора, кроме того, через который он был получен. Важно видеть и понимать работу коммутатора и то, как он осуществляет передачу данных по сети. Понимание функционала коммутатора особенно важно для сетевых администраторов, задача которых заключается в обеспечении безопасной и стабильной работы сети.

Коммутаторы используются для соединения компьютеров в локальных сетях (LAN) и передачи данных между ними. Коммутаторы отправляют кадры Ethernet на узловые устройства, которые идентифицируются по МАС-адресам сетевых плат.
В части 1 необходимо построить топологию, состоящую из двух коммутаторов, соединенных транком. В части 2 необходимо отправить эхо-запросы различным устройствам и посмотреть, как два коммутатора строят свои таблицы МАС-адресов.
Примечание. В лабораторной работе используются коммутаторы Cisco Catalyst 2960s с операционной системой Cisco IOS 15.2(2) (образ lanbasek9).

**Примечание**: Необходимо убедиться, что все настройки коммутатора удалены и загрузочная конфигурация отсутствует. Если вы не уверены в этом, обратитесь к инструктору.

### Ресурсы

Cisco packet tracer с эмулированными:
+ 2 коммутатора (Cisco 2960 с операционной системой Cisco IOS 15.2(2) (образ lanbasek9) или аналогичная модель)
+ 2 ПК (Windows и программа эмуляции терминала, такая как Tera Term)
+ Консольные кабели для настройки устройств Cisco IOS через консольные порты.
+ Кабели Ethernet, расположенные в соответствии с топологией

**Примечание.** Интерфейсы Fast Ethernet на коммутаторах Cisco 2960 определяют тип подключения автоматически, поэтому между коммутаторами S1 и S2 можно использовать прямой кабель Ethernet. При использовании коммутатора Cisco другой модели может потребоваться перекрестный кабель Ethernet.

### Инструкции
### Часть 1. Создание и настройка сети

#### Шаг 1. Подключение сети в соответствии с топологией.

![](/Lab2/mages/00-topology.jpg)

#### Шаг 2. Настройка узлов ПК.

Настройка PC-A:

![](/Lab2/Images/01-PC-A.jpg)

Настройка PC-B:

![](/Lab2/Images/02-PC-B.jpg)

#### Шаг 3. Выполнение инициализации и перезагрузки коммутаторов.

Устанавливаю подключение через консольный порт от PC-A к первому коммутатору, и изучаю конфигурацию:

```
Switch>enable
Switch#show sta
Switch#show startup-config 
startup-config is not present
Switch#
Switch#
Switch#show runn
Switch#show running-config
Building configuration...

Current configuration : 1080 bytes
!
version 15.0
no service timestamps log datetime msec
no service timestamps debug datetime msec
no service password-encryption
!
hostname Switch
!
!
!
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
!
!
!
line con 0
!
line vty 0 4
 login
line vty 5 15
 login
!
!
!
!
end
```

Презагружаю коммутатор, подтверждая командой `Y`:

```
Switch#reload
Proceed with reload? [confirm]y
```

И дожидаюсь завершения перезагрузки:

```
Proceed with reload? [confirm]yC2960 Boot Loader (C2960-HBOOT-M) Version 12.2(25r)FX, RELEASE SOFTWARE (fc4)
Cisco WS-C2960-24TT (RC32300) processor (revision C0) with 21039K bytes of memory.
2960-24TT starting...
Base ethernet MAC Address: 00D0.BA84.B68A
Xmodem file system is available.
Initializing Flash...
flashfs[0]: 1 files, 0 directories
flashfs[0]: 0 orphaned files, 0 orphaned directories
flashfs[0]: Total bytes: 64016384
flashfs[0]: Bytes used: 4670455
flashfs[0]: Bytes available: 59345929
flashfs[0]: flashfs fsck took 1 seconds.
...done Initializing Flash.

Boot Sector Filesystem (bs:) installed, fsid: 3
Parameter Block Filesystem (pb:) installed, fsid: 4


Loading "flash:/2960-lanbasek9-mz.150-2.SE4.bin"...
########################################################################## [OK]
```

После чего коммутатор отображает информацию о себе.

Повторяю процедуру для второго коммутатора

#### Шаг 4. Настройка базовых параметров каждого коммутатора.

Необходимо открыть окно конфигурации, и далее:

a. Настроить имёна устройств в соответствии с топологией

Для первого коммутатора
```
Switch>
Switch>enable
Switch#
Switch#
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#
Switch(config)#
Switch(config)#
Switch(config)#hostname S1
S1(config)#
S1(config)#
```

И для второго коммутатора
```
Switch>
Switch>
Switch>enable
Switch#
Switch#
Switch#
Switch#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#
Switch(config)#
Switch(config)#
Switch(config)#
Switch(config)#hostname S2
S2(config)#
S2(config)#
```

b. Настроить IP-адресов, как указано в таблице адресации:

Для первого коммутатора
```
S1(config)#
S1(config)#int
S1(config)#interface vlan1
S1(config-if)#
S1(config-if)#
S1(config-if)#ip add
S1(config-if)#ip address 192.168.1.11 255.255.255.0
S1(config-if)#
S1(config-if)#
S1(config-if)#no shutdown

S1(config-if)#
%LINK-5-CHANGED: Interface Vlan1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

S1(config-if)#
```

И для второго коммутатора
```
S2(config)#
S2(config)#in
S2(config)#interface vlan1
S2(config-if)#
S2(config-if)#
S2(config-if)#ip add
S2(config-if)#ip address 192.168.1.12 255.255.255.0
S2(config-if)#
S2(config-if)#
S2(config-if)#
S2(config-if)#no shutdown

S2(config-if)#
%LINK-5-CHANGED: Interface Vlan1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

S2(config-if)#
```

c. Назначить `cisco` в качестве паролей консоли и VTY.

Для первого коммутатора
```
S1(config)#
S1(config)#line console 0
S1(config-line)#
S1(config-line)#pass
S1(config-line)#password cisco
S1(config-line)#
S1(config-line)#login
S1(config)#
S1(config)#
S2(config-line)#exit
S1(config)#
S1(config)#
S1(config)#
S1(config)#line vty 0 15
S1(config-line)#
S1(config-line)#pass
S1(config-line)#password cisco
S1(config-line)#
S1(config-line)#login
S1(config-line)#
S1(config-line)#exit
S1(config)#
S1(config)#

```

И для второго коммутатора
```
S2(config)#
S2(config)#line console 0
S2(config-line)#
S2(config-line)#pas
S2(config-line)#password cisco
S2(config-line)#
S2(config-line)#login
S2(config-line)#
S2(config-line)#
S2(config-line)#exit
S2(config)#
S2(config)#
S2(config)#line vty 0 15
S2(config-line)#
S2(config-line)#pass
S2(config-line)#password cisco
S2(config-line)#
S2(config-line)#login
S2(config-line)#
S2(config-line)#
S2(config-line)#exit
S2(config)#
S2(config)#

```

d. Назначить `class` в качестве пароля доступа к привилегированному режиму EXEC.

Для первого коммутатора
```
S1(config)#
S1(config)#enable secret class
S1(config)#
S1(config)#
```

И для второго коммутатора
```
S2(config)#
S2(config)#enable secret class
S2(config)#
S2(config)#
```

### Часть 2. Изучение таблицы МАС-адресов коммутатора

Как только между сетевыми устройствами начинается передача данных, коммутатор выясняет МАС-адреса и строит таблицу.

#### Шаг 1. Записать МАС-адреса сетевых устройств.

a. Необходимо открыть командную строку на PC-A и PC-B и ввести команду `ipconfig /all`.

Открытие окна командной строки Windows:

![](/Lab2/Images/03-ipconfig.jpg)

Вопросы:

*Назовите физические адреса адаптера Ethernet.*

*MAC-адрес компьютера PC-A:* 0003.E492.916E

*MAC-адрес компьютера PC-B:* 0009.7c0c.0797

Теперь необходимо закрыть окно командной строки.

b.	Необходимо подключиться к коммутаторам S1 и S2 через консоль и ввести команду `show interface F0/1` на каждом коммутаторе.

Открытие окна конфигурации

Вопросы:

*Назовите адреса оборудования во второй строке выходных данных команды (или зашитый адрес — bia).*

*МАС-адрес коммутатора S1 Fast Ethernet 0/1:* 
`Hardware is Lance, address is 0001.4391.d001 (bia 0001.4391.d001)`

*МАС-адрес коммутатора S2 Fast Ethernet 0/1:*
`Hardware is Lance, address is 00d0.ff4e.1501 (bia 00d0.ff4e.1501)`


#### Шаг 2. Просмотр таблицу МАС-адресов коммутатора.

Необходимо подключиться к коммутатору S2 через консоль и просмотрите таблицу МАС-адресов до и после тестирования сетевой связи с помощью эхо-запросов.

a. Подключение к коммутатору S2 через консоль и вход в привилегированный режим EXEC.

```
S2>
S2>
S2>enable
Password: 
S2#
S2#
S2#
```

b. В привилегированном режиме EXEC необходимо ввести команду `show mac address-table` и нажать клавишу ввода.

`S2# show mac address-table`


```
S2#
S2#show mac add
S2#show mac address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.4391.d001    DYNAMIC     Fa0/1
S2#
```

Даже если сетевая коммуникация в сети не происходила (т. е. если команда ping не отправлялась), коммутатор может узнать МАС-адреса при подключении к ПК и другим коммутаторам.

Вопросы:

*Записаны ли в таблице МАС-адресов какие-либо МАС-адреса?* 

`0001.4391.d001`

*Какие МАС-адреса записаны в таблице? С какими портами коммутатора они сопоставлены и каким устройствам принадлежат? Игнорируйте МАС-адреса, сопоставленные с центральным процессором.*

МАС-адрес коммутатора S1, он принадлежит порту FastEthernet 0/1

*Если вы не записали МАС-адреса сетевых устройств в шаге 1, как можно определить, каким устройствам принадлежат МАС-адреса, используя только выходные данные команды show mac address-table? Работает ли это решение в любой ситуации?*

С помощью команды `show mac address-table`  коммутатор отобразит те порты, для которых были получены MAC-адреса. В ряде случаев этого окажется достаточно, кроме случаев, когда коммутаторы подключены к другим коммутаторам, за которыми будут видны MAC-адреса всех подключенных устройств.

#### Шаг 3. Очистка таблицы МАС-адресов коммутатора S2 и новое отображение таблицы это

a.	В привилегированном режиме EXEC введите команду clear mac address-table dynamic и нажмите клавишу Enter.

`S2# clear mac address-table dynamic`

```
S2>
S2>
S2>enable
Password: 
S2#
S2#
S2#clear mac ad
S2#clear mac address-table dynamic
S2#
```

b.	Снова быстро введите команду show mac address-table.
```
S2#
S2#
S2#show
S2#show mac add
S2#show mac address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

S2#
S2#
```
Вопросы:

*Указаны ли в таблице МАС-адресов адреса для VLAN 1? Указаны ли другие МАС-адреса?*

В этом случае, как видно в листинге командной строки, таблица адресов пуста.

*Через 10 секунд введите команду show mac address-table и нажмите клавишу ввода. Появились ли в таблице МАС-адресов новые адреса?*

```
S2#
S2#show mac address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.4391.d001    DYNAMIC     Fa0/1
S2#
```

Как видно из листинга, спустя 10 секунд после очистки таблицы MAC-адресов, в ней появился физический адрес коммутатора S1

#### Шаг 4. Отправка с компьютера PC-B эхо-запросов устройствам в сети и просмотр таблицы МАС-адресов коммутатора.

a.	На компьютере PC-B необходимо открыть командную строку и ввести команду `arp -a`.
```
C:\>
C:\>arp -a
No ARP Entries Found
C:\>
```

*Не считая адресов многоадресной и широковещательной рассылки, сколько пар IP- и МАС-адресов устройств было получено через протокол ARP?*

Адреса не были получены


b.	Из командной строки PC-B необходимо отправить эхо-запросы на компьютер PC-A, а также коммутаторы S1 и S2.

```
C:\>
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=128
Reply from 192.168.1.1: bytes=32 time=3ms TTL=128
Reply from 192.168.1.1: bytes=32 time<1ms TTL=128
Reply from 192.168.1.1: bytes=32 time<1ms TTL=128

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 3ms, Average = 0ms

C:\>ping 192.168.1.11

Pinging 192.168.1.11 with 32 bytes of data:

Request timed out.
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255
Reply from 192.168.1.11: bytes=32 time<1ms TTL=255
Reply from 192.168.1.11: bytes=32 time=2ms TTL=255

Ping statistics for 192.168.1.11:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 2ms, Average = 0ms

C:\>ping 192.168.1.12

Pinging 192.168.1.12 with 32 bytes of data:

Request timed out.
Reply from 192.168.1.12: bytes=32 time<1ms TTL=255
Reply from 192.168.1.12: bytes=32 time<1ms TTL=255
Reply from 192.168.1.12: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.12:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

*От всех ли устройств получены ответы? Если нет, проверьте кабели и IP-конфигурации.*

Ответы, как видно из листинга командной строки, получены ото всех устройств

c.	Подключившись через консоль к коммутатору S2, необходимо ввести команду `show mac address-table`.

```
S2#
S2#
S2#show mac
S2#show mac-ad
S2#show mac-address-table 
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----

   1    0001.4391.d001    DYNAMIC     Fa0/1 
   1    0003.e492.916e    DYNAMIC     Fa0/1
   1    0009.7c0c.0797    DYNAMIC     Fa0/18 
   1    00d0.ba84.b68a    DYNAMIC     Fa0/1 
S2#
S2#
```

*Добавил ли коммутатор в таблицу МАС-адресов дополнительные МАС-адреса? Если да, то какие адреса и устройства?*

Как видно из листинга терминала коммутатора S2, после эхо запросов стали видны физические адреса:

+ порта 1 FastEthernet коммутатора S1
+ компьютера PC-A за коммутатором S1
+ компьютера PC-B на 18 порте FastEthernet
+ адрес коммутатора S1

На компьютере PC-B необходимо открыть командную строку и еще раз ввести команду `arp -a`.

```
C:\>
C:\>arp -a
  Internet Address      Physical Address      Type
  192.168.1.1           0003.e492.916e        dynamic
  192.168.1.11          00d0.ba84.b68a        dynamic
  192.168.1.12          0050.0f41.5b72        dynamic

C:\>
```
**Примечание** Физический адресс `0050.0f41.5b72` в arp-таблице, отображаемой на PC-B является собственным адресом коммутатора S2. Отображается, например, при выполнении команды `S2#show arp`

```
S2#
S2#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.1.2             0   0009.7C0C.0797  ARPA   Vlan1
Internet  192.168.1.12            -   0050.0F41.5B72  ARPA   Vlan1
S2#
```

*Появились ли в ARP-кэше компьютера PC-B дополнительные записи для всех сетевых устройств, которым были отправлены эхо-запросы?*

В ARP-кэше компьютера PC-B дополнительные записи появились для всех устройств - это видно из листинга командной строки.

#### Вопрос для повторения

*В сетях Ethernet данные передаются на устройства по соответствующим МАС-адресам. Для этого коммутаторы и компьютеры динамически создают ARP-кэш и таблицы МАС-адресов. Если компьютеров в сети немного, эта процедура выглядит достаточно простой. Какие сложности могут возникнуть в крупных сетях?*

ARP-запросы являются широковещательными, соответственно в сетях, где окно ожидания получения пакета данных невелико, могут возникнуть коллизии.
К тому же ARP-таблица и таблица MAC-адресов коммутатора не проверяют соответствие MAC- и IP-адресов - возможна подмена устройств
