## Лабораторная работа - Реализация DHCPv4  

### Топология

![](/Lab8_1/Images/00_topology.jpg)

### 1. Таблица адресации 
Устройство | Интерфейс   | IP-адрес      | Маска подсети   | Шлюз по умолчанию |
-----------|-------------|---------------|-----------------|-------------------|
R1         | G0/0/0      | 10.0.0.1      | 255.255.255.252 | -                 |
|          | G0/0/1      | -             | -               | -                 |
|          | G0/0/1.100  | 192.168.1.1   | 255.255.255.192 | -                 |
|          | G0/0/1.200  | 192.168.1.65  | 255.255.255.224 | -                 |
|          | G0/0/1.1000 | -             | -               | -                 |
R2         | G0/0        | 10.0.0.2      | 255.255.255.0   | -                 |
|          | G0/0/1      | 192.168.1.97  | 255.255.255.240 | -                 |
S1         | VLAN 200    | 192.168.1.66  | 255.255.255.224 | 192.168.1.65      |
S2         | VLAN 1      | 192.168.1.98  | 255.255.255.240 | 192.168.1.97      |
PC-A       | NIC         | DHCP          | DHCP            | DHCP              |
PC-B       | NIC         | DHCP          | DHCP            | DHCP              |

### 2. Таблица VLAN
VLAN | Имя         | Назначенный нтерфейс        |
-----|-------------|-----------------------------|
1    | Нет         | S2: F0/18                   |
100  | Клиенты     | S1: F0/6                    |
200  | Управление  | S1: VLAN 200                |
999	 | Parking_Lot | S1: F0/1-4, F0/7-24, G0/1-2 |
1000 | Собственная | —                           |

### 3. Задачи
#### Часть 1. Создание сети и настройка основных параметров устройства
#### Часть 2. Настройка и проверка двух серверов DHCPv4 на R1
#### Часть 3. Настройка и проверка DHCP-ретрансляции на R2

### Общие сведения/сценарий

Протокол динамической конфигурации сетевого узла (DHCP) — сетевой протокол, позволяющий сетевым администраторам управлять и автоматизировать назначение IP-адресов. Без использования DHCP  для IPv4 администратору необходимо вручную назначать и настраивать IP-адреса, предпочтительные DNS-серверы и шлюзы по умолчанию. По мере увеличения сети и перемещении устройств из одной внутренней сети в другую это становится административной проблемой.

В предложенном сценарии размеры компании увеличились, и сетевые администраторы больше не имеют возможности назначать IP-адреса для устройств вручную. Задача лабораторной работы заключается в настройке маршрутизатора R1 для назначения IPv4-адресов в двух разных подсетях. 

**Примечание:** Маршрутизаторы, используемые в практических лабораторных работах CCNA — это Cisco 4221 с Cisco IOS XE Release 16.9.4 (образ universalk9). В лабораторных работах используются коммутаторы Cisco Catalyst 2960 с Cisco IOS версии 15.2(2) (образ lanbasek9). Можно использовать другие маршрутизаторы, коммутаторы и версии Cisco IOS. В зависимости от модели устройства и версии Cisco IOS доступные команды и результаты их выполнения могут отличаться от тех, которые показаны в лабораторных работах. Правильные идентификаторы интерфейса см. в сводной таблице по интерфейсам маршрутизаторов в конце лабораторной работы.

**Примечание:** Необходимо убедиться, что у всех маршрутизаторов и коммутаторов была удалена начальная конфигурация.

### Необходимые ресурсы

+ 2 маршрутизатора (Cisco 4221 с универсальным образом Cisco IOS XE версии 16.9.4 или аналогичным)
+ 2 коммутатора (Cisco 2960 с операционной системой Cisco IOS 15.2(2) (образ lanbasek9) или аналогичная модель)
+ 2 ПК (ОС Windows с программой эмуляции терминалов, такой как Tera Term)
+ Консольные кабели для настройки устройств Cisco IOS через консольные порты.
+ Кабели Ethernet, расположенные в соответствии с топологией

### Инструкции

### Часть 1. Создание сети и настройка основных параметров устройства

В первой части лабораторной работы предстоит создать топологию сети и настроить базовые параметры для узлов ПК и коммутаторов.

#### Шаг 1. Создание схемы адресации

Подсеть сети 192.168.1.0/24 в соответствии со следующими требованиями:

a. Одна подсеть «Подсеть A», поддерживающая 58 хостов (клиентская VLAN на R1).

Подсеть A:

`192.168.1.0/26 (192.168.1.1 - 192.168.1.63)`

Необходимо записать первый IP-адрес в таблице адресации для R1 G0/0/1.100 . 

b. Одна подсеть «Подсеть B», поддерживающая 28 хостов (управляющая VLAN на R1).

Подсеть B:

`192.168.1.64/27 (192.168.1.65 - 192.168.1.95)`

Необходимо записать первый IP-адрес в таблице адресации для R1 G0/0/1.200. Необходимо записать второй IP-адрес в таблице адресов для S1 VLAN 200 и введите соответствующий шлюз по умолчанию.

c. Одна подсеть «Подсеть C», поддерживающая 12 узлов (клиентская сеть на R2).

Подсеть C:

`192.168.1.96/28 (192.168.1.97 - 192.168.1.111)`

Необходимо записать первый IP-адрес в таблице адресации для R2 G0/0/1.

#### Шаг 2. Создание сеть согласно топологии.

Необходимо подключить устройства, как показано в топологии, и подсоедините необходимые кабели.

#### Шаг 3. Базовая настройка маршрутизаторов.

a. Необходимо назначить маршрутизатору имя устройства.

Необходимо открыть окно конфигурации

b. Необходимо отключить поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

c. Необходимо назначить class в качестве зашифрованного пароля привилегированного режима EXEC.

d. Необходимо назначить cisco в качестве пароля консоли и включите вход в систему по паролю.

e. Необходимо назначить cisco в качестве пароля VTY и включите вход в систему по паролю.

f. Необходимо зашифровать открытые пароли.

g. Необходимо создать баннер с предупреждением о запрете несанкционированного доступа к устройству.

h. Необходимо сохранить текущую конфигурацию в файл загрузочной конфигурации.

i. Необходимо установить часы на маршрутизаторе на сегодняшнее время и дату.

**Примечание.** Вопросительный знак (?) позволяет открыть справку с правильной последовательностью параметров, необходимых для выполнения этой команды.

#### Шаг 4. Настройка маршрутизации между сетями VLAN на маршрутизаторе R1

a. Необходимо активировать интерфейс G0/0/1 на маршрутизаторе.

```
R1(config)# interface g0/0/1
R1(config-if)# no shutdown
R1(config-if)# exit
```

b. Необходимо настроить подинтерфейсы для каждой VLAN в соответствии с требованиями таблицы IP-адресации. Все субинтерфейсы используют инкапсуляцию 802.1Q и назначаются первый полезный адрес из вычисленного пула IP-адресов. Убедитесь, что подинтерфейсу для native VLAN не назначен IP-адрес. Включите описание для каждого подинтерфейса.

```
R1(config)#
R1(config)#interface g0/0/1.100
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.100, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.100, changed state to up

R1(config-subif)#
R1(config-subif)#
R1(config-subif)#descr
R1(config-subif)#description CLIENT NETWORK
R1(config-subif)#
R1(config-subif)#
R1(config-subif)#enc
R1(config-subif)#encapsulation dot1q 100
R1(config-subif)#
R1(config-subif)#
R1(config-subif)#
R1(config-subif)#ip add
R1(config-subif)#ip address 192.168.1.1 255.255.255.192
R1(config-subif)#
R1(config-subif)#
R1(config-subif)#
R1(config-subif)#
R1(config-subif)#interface g0/0/1.200
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.200, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.200, changed state to up

R1(config-subif)#encapsulation dot1q 200
R1(config-subif)#
R1(config-subif)#
R1(config-subif)#description MANAGEMENT NETWORK
R1(config-subif)#ip address 192.168.1.65 255.255.255.224
R1(config-subif)#interface g0/0/1.1000
R1(config-subif)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1.1000, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1.1000, changed state to up

R1(config-subif)#
R1(config-subif)#
R1(config-subif)#encapsulation dot1q 1000 native
R1(config-subif)#description Native VLAN
```

c. Необходимо убедиться, что вспомогательные интерфейсы работают.

```
R1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol 
GigabitEthernet0/0/0   unassigned      YES unset  administratively down down 
GigabitEthernet0/0/1   unassigned      YES unset  up                    up 
GigabitEthernet0/0/1.100192.168.1.1     YES manual up                    up 
GigabitEthernet0/0/1.200192.168.1.65    YES manual up                    up 
GigabitEthernet0/0/1.1000unassigned      YES unset  up                    up 
GigabitEthernet0/0/2   unassigned      YES unset  administratively down down 
Vlan1                  unassigned      YES unset  administratively down down
```

#### Шаг 5. Настройка G0/1 на R2, затем G0/0/0 и статическую маршрутизацию для обоих маршрутизаторов

a. Необходимо настроить G0/0/1 на R2 с первым IP-адресом подсети C, рассчитанным ранее.

```
R2(config)#interface  G0/0/1
R2(config-if)#
R2(config-if)#
R2(config-if)#ip add
R2(config-if)#ip address 192.168.1.97 255.255.255.240
R2(config-if)#
R2(config-if)#
R2(config-if)#
R2(config-if)#no shu
R2(config-if)#no shutdown 

R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/1, changed state to up

R2(config-if)#
R2(config-if)#
R2(config-if)#exit
```

b. Необходимо настроить интерфейс G0/0/0 для каждого маршрутизатора на основе приведенной выше таблицы IP-адресации.

```
R1(config)#interface g0/0/0
R1(config-if)#
R1(config-if)#ip address 10.0.0.1 255.255.255.252
R1(config-if)#
R1(config-if)#
R1(config-if)#
R1(config-if)#no shut
R1(config-if)#no shutdown 

R1(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0/0, changed state to up
```

```
R2(config)#interface G0/0/0
R2(config-if)#
R2(config-if)#ip address 10.0.0.2 255.255.255.252
R2(config-if)#
R2(config-if)#
R2(config-if)#no shut

R2(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/0/0, changed state to up
```

c. Необходимо настроить маршрут по умолчанию на каждом маршрутизаторе, указываемом на IP-адрес G0/0/0 на другом маршрутизаторе.

```
R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

```
R2(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

d. Необходимо убедиться, что статическая маршрутизация работает с помощью пинга до адреса G0/0/1 R2 от R1.

```
R1#ping 192.168.1.97

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.97, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms
```

e. Необходимо сохрать текущую конфигурацию в файл загрузочной конфигурации.

```
R1#copy running-config start
R1#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
R1#
```

```
R2#copy runn
R2#copy running-config start
R2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
```

#### Шаг 6. Настройка базовых параметров каждого коммутатора.

a. Необходимо присвоить коммутатору имя устройства.

Необходимо открыть окно конфигурации

b. Необходимо отключить поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

c. Необходимо назначить class в качестве зашифрованного пароля привилегированного режима EXEC.

d. Необходимо назначить cisco в качестве пароля консоли и включите вход в систему по паролю.

e. Необходимо назначить cisco в качестве пароля VTY и включите вход в систему по паролю.

f. Необходимо зашифровать открытые пароли.

g. Необходимо создать баннер с предупреждением о запрете несанкционированного доступа к устройству.

h. Необходимо сохранить текущую конфигурацию в файл загрузочной конфигурации.

i. Необходимо установить часы на маршрутизаторе на сегодняшнее время и дату.

**Примечание:** Вопросительный знак (?) позволяет открыть справку с правильной последовательностью параметров, необходимых для выполнения этой команды.

j. Необходимо скопировать текущую конфигурацию в файл загрузочной конфигурации

#### Шаг 7. Создание сети VLAN на коммутаторе S1.

**Примечание:** S2 настроен только с базовыми настройками. 

a. Необходимо создать необходимые VLAN на коммутаторе 1 и присвойте им имена из приведенной выше таблицы.

```
S1(config)#vlan 100
S1(config-vlan)#
S1(config-vlan)#name CLIENTS
S1(config-vlan)#
S1(config-vlan)#vlan 200
S1(config-vlan)#
S1(config-vlan)#name MANAGEMENT
S1(config-vlan)#
S1(config-vlan)#vlan 999
S1(config-vlan)#
S1(config-vlan)#name Parking_Lot
S1(config-vlan)#
S1(config-vlan)#vlan 1000
S1(config-vlan)#
S1(config-vlan)#name Native
S1(config-vlan)#exit
```

b. Необходимо настроить и активировать интерфейс управления на S1 (VLAN 200), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того необходимо установить шлюз по умолчанию на S1.

```
S1(config)#interface vlan 200
S1(config-if)#
%LINK-5-CHANGED: Interface Vlan200, changed state to up

S1(config-if)#
S1(config-if)#ip address 192.168.1.66 255.255.255.224
S1(config-if)#
S1(config-if)#no shutdown
S1(config-if)#
S1(config-if)#
S1(config-if)#exit
S1(config)#
S1(config)#
S1(config)#ip def
S1(config)#ip default-gateway 192.168.1.65
```

c. Необходимо настроить и активировать интерфейс управления на S2 (VLAN 1), используя второй IP-адрес из подсети, рассчитанный ранее. Кроме того, необходимо установить шлюз по умолчанию на S2

```
S2(config)#interface vlan 1
S2(config-if)#
S2(config-if)#
S2(config-if)#
S2(config-if)#ip address 192.168.1.98 255.255.255.240
S2(config-if)#
S2(config-if)#
S2(config-if)#no shu
S2(config-if)#no shutdown 

S2(config-if)#
%LINK-5-CHANGED: Interface Vlan1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up

S2(config-if)#
S2(config-if)#exit 
S2(config)#
S2(config)#
S2(config)#ip def
S2(config)#ip default-gateway 192.168.1.97
S2(config)#
```

d. Необходимо назначить все неиспользуемые порты S1 VLAN Parking_Lot, настройте их для статического режима доступа и административно деактивируйте их. На S2 необходимо административно дективировать все неиспользуемые порты.

```
S1(config)#int
S1(config)#interface ra
S1(config)#interface range F0/1 - 4, f0/7 - 24, g0/1 - 2
S1(config-if-range)#swi
S1(config-if-range)#switchport mode access
S1(config-if-range)#
S1(config-if-range)#sw
S1(config-if-range)#switchport acc
S1(config-if-range)#switchport access vlan 999
S1(config-if-range)#
S1(config-if-range)#shutdown

%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/7, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/8, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/9, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/10, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/11, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/12, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/13, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/14, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/15, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/16, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/17, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/18, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/19, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/20, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/21, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/22, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/23, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/24, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
```

```
S2(config)#int
S2(config)#interface ra
S2(config)#interface range f0/1 - 4, f0/6 - 17, f0/19 - 24, g0/1 - 2
S2(config-if-range)#swit
S2(config-if-range)#switchport mode access
S2(config-if-range)#shut
S2(config-if-range)#shutdown 

%LINK-5-CHANGED: Interface FastEthernet0/1, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/2, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/3, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/4, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/6, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/7, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/8, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/9, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/10, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/11, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/12, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/13, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/14, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/15, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/16, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/17, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/19, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/20, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/21, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/22, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/23, changed state to administratively down

%LINK-5-CHANGED: Interface FastEthernet0/24, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down

%LINK-5-CHANGED: Interface GigabitEthernet0/2, changed state to administratively down
```

**Примечание:** Команда `interface range` полезна для выполнения этой задачи с минимальным количеством команд.

#### Шаг 8.	Назначение сети VLAN соответствующим интерфейсам коммутатора.

a. Необходимо назначить используемые порты соответствующей VLAN (указанной в таблице VLAN выше) и настройте их для режима статического доступа.

Необходимо открыть окно конфигурации

```
S1(config)#interface f0/6
S1(config-if)#
S1(config-if)#
S1(config-if)#swi
S1(config-if)#switchport mode acce
S1(config-if)#switchport mode access 
S1(config-if)#switchport acc
S1(config-if)#switchport access vlan 100
```

b. Необходимо убедиться, что VLAN назначены на правильные интерфейсы.

```
S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/5
100  CLIENTS                          active    Fa0/6
200  MANAGEMENT                       active    
999  Parking_Lot                      active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
1000 Native                           active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active    
```

**Вопрос:**

Почему интерфейс F0/5 указан в VLAN 1?

**Ответ:**

Порт 5 находится в сети VLAN по умолчанию и не был настроен как магистральный порт 802.1Q.

#### Шаг 9.	Ручная настройка интерфейса S1 F0/5 в качестве транка 802.1Q.

a. Необходимо изменить режим порта коммутатора, чтобы принудительно создать магистральный канал.

```
S1(config)#int
S1(config)#interface f0/5
S1(config-if)#
S1(config-if)#
S1(config-if)#switch
S1(config-if)#switchport mode trunk

S1(config-if)#
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/5, changed state to down

%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/5, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan200, changed state to up

```
b. В рамках конфигурации транка необходимо установить для native  VLAN значение 1000.

```
S1(config-if)#switchport trunk native vlan 1000
```

c. В качестве другой части конфигурации магистрали необходимо указать, что VLAN 100, 200 и 1000 могут проходить по транку.

```
S1(config-if)#switchport trunk allowed vlan 100,200,1000
```

d. Необходимо сохранить текущую конфигурацию в файл загрузочной конфигурации.

e. Необходимо проверить состояние транка.

```
S1#show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Fa0/5       on           802.1q         trunking      1000

Port        Vlans allowed on trunk
Fa0/5       100,200,1000

Port        Vlans allowed and active in management domain
Fa0/5       100,200,1000

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/5       100,200,1000
```

**Вопрос:**

Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP?

**Ответ:**

Они будут автоматически настраиваться с помощью адреса частного IP-адреса (APIPA) в диапазоне 169.254.x.x

### Часть 2. Настройка и проверка двух серверов DHCPv4 на R1

В части 2 необходимо настроить и проверить сервер DHCPv4 на R1. Сервер DHCPv4 будет обслуживать две подсети, подсеть A и подсеть C.

#### Шаг 1. Настройка R1 с пулами DHCPv4 для двух поддерживаемых подсетей. Ниже приведен только пул DHCP для подсети A

a.	Необходимо исключить первые пять используемых адресов из каждого пула адресов.

Необходимо открыть окно конфигурации

```
R1(config)#
R1(config)#ip dhcp ex
R1(config)#ip dhcp excluded-address 192.168.1.1 192.168.1.5
```

b. Необходимо создать пул DHCP (используйте уникальное имя для каждого пула).

```
R1(config)#ip dhcp pool R1_Client_LAN
```

c. Необходимо указать сеть, поддерживающую этот DHCP-сервер.

```
R1(dhcp-config)#network 192.168.1.0 255.255.255.192
```

d. В качестве имени домена необходимо указать CCNA-lab.com.

```
R1(dhcp-config)#domain-name OTUS.com
```

e. Необходимо настроить соответствующий шлюз по умолчанию для каждого пула DHCP.

```
R1(dhcp-config)#def
R1(dhcp-config)#default-router 192.168.1.1
```

f. Необходимо настроить время аренды на 2 дня 12 часов и 30 минут.

```
PT не позволил указать срок публикации
```

g. Затем необходимо настроить второй пул DHCPv4, используя имя пула R2_Client_LAN, и вычислить сеть, маршрутизатор по умолчанию, и использовать то же имя домена и время аренды, что и предыдущий пул DHCP.

```
R1(config)#ip dhcp excl
R1(config)#ip dhcp excluded-address 192.168.1.97 192.168.1.101
R1(config)#
R1(config)#
R1(config)#
R1(config)#IP dhcp pool R2_Client_LAN
R1(dhcp-config)#
R1(dhcp-config)#
R1(dhcp-config)#
R1(dhcp-config)#netw
R1(dhcp-config)#network 192.168.1.96 255.255.255.240
R1(dhcp-config)#
R1(dhcp-config)#
R1(dhcp-config)#def
R1(dhcp-config)#default-router 192.168.1.97
R1(dhcp-config)#
R1(dhcp-config)#
R1(dhcp-config)#domaint
R1(dhcp-config)#domain-nam
R1(dhcp-config)#domain-name OTUS.com
R1(dhcp-config)#
R1(dhcp-config)#
R1(dhcp-config)#
R1(dhcp-config)#end
```
#### Шаг 2.	Сохранение конфигурацию.

Необходимо сохранить текущую конфигурацию в файл загрузочной конфигурации.

#### Шаг 3.	Проверка конфигурации сервера DHCPv4

a. Чтобы просмотреть сведения о пуле, необходимо выполнить команду `show ip dhcp pool`.

```
Pool R1_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 62
 Leased addresses               : 1
 Excluded addresses             : 2
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.1          192.168.1.1      - 192.168.1.62      1    / 2     / 62

Pool R2_Client_LAN :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 14
 Leased addresses               : 0
 Excluded addresses             : 2
 Pending event                  : none

 1 subnet is currently in the pool
 Current index        IP address range                    Leased/Excluded/Total
 192.168.1.97         192.168.1.97     - 192.168.1.110     0    / 2     / 14
```

b. Необходимо выполнить команду `show ip dhcp bindings` для проверки установленных назначений адресов DHCP.

```
R1#show ip dhcp bindings
                       ^
% Invalid input detected at '^' marker.
	
R1#show ip dhcp binding
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0007.ECBD.A974           --                     Automatic
```

c. Необходимо выполнить команду `show ip dhcp server statistics` для проверки сообщений DHCP.

#### Шаг 4. Попытка получения IP-адреса от DHCP на PC-A

a. Из командной строки компьютера PC-A необходимо выполнить команду `ipconfig /all`.

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::207:ECFF:FEBD:A974
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: ::
                                     0.0.0.0
```

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: OTUS.com
   Physical Address................: 0007.ECBD.A974
   Link-local IPv6 Address.........: FE80::207:ECFF:FEBD:A974
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: ::
                                     192.168.1.1
   DHCP Servers....................: 192.168.1.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-E2-8D-52-EB-00-07-EC-BD-A9-74
   DNS Servers.....................: ::
                                     0.0.0.0
```

b. После завершения процесса обновления необходимо выполнить команду `ipconfig` для просмотра новой информации об IP-адресе.

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: OTUS.com
   Link-local IPv6 Address.........: FE80::207:ECFF:FEBD:A974
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.6
   Subnet Mask.....................: 255.255.255.192
   Default Gateway.................: ::
                                     192.168.1.1
```

c. Необходимо проверить подключение с помощью пинга IP-адреса интерфейса R0 G0/0/1.

```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
```

### Часть 3. Настройка и проверка DHCP-ретрансляции на R2

В части 3 настраивается R2 для ретрансляции DHCP-запросов из локальной сети на интерфейсе G0/0/1 на DHCP-сервер (R1). 

#### Шаг 1.	Настройка R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1

a. Необходимо настроить команду `ip helper-address` на G0/0/1, указав IP-адрес G0/0/0 R1.

Необходимо открыть окно конфигурации

```
R2(config-if)#ip helper-address 10.0.0.1
```

b. Необходимо сохранить конфигурацию.

```
R2#copy running-config st
R2#copy running-config startup-config 
Destination filename [startup-config]? 
Building configuration...
[OK]
```
#### Шаг 2. Попытка получения IP-адреса от DHCP на PC-B

a. Из командной строки компьютера PC-B необходимо выполнить команду `ipconfig /all`.

```
C:\>ipconfig /all

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: OTUS.com
   Physical Address................: 000A.F3E9.7311
   Link-local IPv6 Address.........: FE80::20A:F3FF:FEE9:7311
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.102
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: ::
                                     192.168.1.97
   DHCP Servers....................: 10.0.0.1
   DHCPv6 IAID.....................: 
   DHCPv6 Client DUID..............: 00-01-00-01-E2-8D-52-EB-00-0A-F3-E9-73-11
   DNS Servers.....................: ::
                                     0.0.0.0
```

b. После завершения процесса обновления необходимо выполнить команду `ipconfig` для просмотра новой информации об IP-адресе.

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: OTUS.com
   Link-local IPv6 Address.........: FE80::20A:F3FF:FEE9:7311
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.102
   Subnet Mask.....................: 255.255.255.240
   Default Gateway.................: ::
                                     192.168.1.97
```

c. Необходимо проверить подключение с помощью пинга IP-адреса интерфейса R1 G0/0/1.

```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254
Reply from 192.168.1.1: bytes=32 time<1ms TTL=254

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms

```

d. Необходимо выполнить `show ip dhcp binding` для R1 для проверки назначений адресов в DHCP

```
R1#show ip dhcp bi
R1#show ip dhcp binding 
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.1.6      0007.ECBD.A974           --                     Automatic
192.168.1.102    000A.F3E9.7311           --                     Automatic
```

e. Необходимо выполнить команду `show ip dhcp server statistics` для проверки сообщений DHCP.

```
R1#
R1#show ip dhcp ser
R1#show ip dhcp server statistics
                ^
% Invalid input detected at '^' marker.
```