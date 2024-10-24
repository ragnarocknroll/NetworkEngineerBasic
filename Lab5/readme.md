## Лабораторная работа №5. Доступ к сетевым устройствам по протоколу SSH

### Топология

![](/Lab5/Images/00_topology.jpg)

### Таблица адресации 

Устройство | Интерфейс | IP-адрес     | Маска подсети | Шлюз по умолчанию |
-----------|-----------|--------------|---------------|-------------------|
R1         | G0/0/1    | 192.168.1.1  | 255.255.255.0 |          -        |
S1         | VLAN 1    | 192.168.1.11 | 255.255.255.0 | 192.168.1.1       |
PC-A       | NIC       | 192.168.1.3  | 255.255.255.0 | 192.168.1.1       |

### Задачи

#### Часть 1. Настройка основных параметров устройства
#### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH
#### Часть 3. Настройка коммутатора для доступа по протоколу SSH
#### Часть 4. SSH через интерфейс командной строки (CLI) коммутатора

### Общие сведения/сценарий

Раньше для удаленной настройки сетевых устройств в основном применялся протокол Telnet. Однако он не обеспечивает шифрование информации, передаваемой между клиентом и сервером, что позволяет анализаторам сетевых пакетов перехватывать пароли и данные конфигурации.

Secure Shell (SSH) — это сетевой протокол, устанавливающий безопасное подключение с эмуляцией терминала к маршрутизатору или иному сетевому устройству. Протокол SSH шифрует все сведения, которые поступают по сетевому каналу, и предусматривает аутентификацию удаленного компьютера. Протокол SSH все больше заменяет Telnet — именно его выбирают сетевые специалисты в качестве средства удаленного входа в систему. SSH чаще всего используется для входа на удаленное устройство и выполнения команд. Но это может также передавать файлы по связанным протоколам SFTP или SCP.

Чтобы протокол SSH мог работать, на сетевых устройствах, взаимодействующих между собой, должна быть настроена поддержка SSH. В этой лабораторной работе необходимо включить SSH-сервер на маршрутизаторе, после чего подключиться к этому маршрутизатору, используя ПК с установленным клиентом SSH. В локальной сети подключение обычно устанавливается с помощью Ethernet и IP.

**Примечание:** Маршрутизаторы, используемые в практических лабораторных работах CCNA, - это Cisco 4221 с Cisco IOS XE Release 16.9.4 (образ universalk9). В лабораторных работах используются коммутаторы Cisco Catalyst 2960 с Cisco IOS версии 15.2(2) (образ lanbasek9). Можно использовать другие маршрутизаторы, коммутаторы и версии Cisco IOS. В зависимости от модели устройства и версии Cisco IOS доступные команды и результаты их выполнения могут отличаться от тех, которые показаны в лабораторных работах. Правильные идентификаторы интерфейса см. в сводной таблице по интерфейсам маршрутизаторов в конце лабораторной работы.

**Примечание:**  Необходимо убедиться, что у всех маршрутизаторов и коммутаторов была удалена начальная конфигурация.

### Необходимые ресурсы

+ 1 Маршрутизатор (Cisco 4221 с универсальным образом Cisco IOS XE версии 16.9.4 или аналогичным)
+ 1 коммутатор (Cisco 2960 с ПО Cisco IOS версии 15.2(2) с образом lanbasek9 или аналогичная модель)
+ 1 ПК (под управлением Windows с программой эмуляции терминала, например, Tera Term)
+ Консольные кабели для настройки устройств Cisco IOS через консольные порты.
+ Кабели Ethernet, расположенные в соответствии с топологией

### Инструкции

### Часть 1. Настройка основных параметров устройств

В части 1 необходимо настроить топологию сети и основные параметры, такие как IP-адреса интерфейсов, доступ к устройствам и пароли на маршрутизаторе.

#### Шаг 1. Необходимо создать сеть согласно топологии.

#### Шаг 2. Необходимо выполнить инициализацию и перезагрузку маршрутизатора и коммутатора.

#### Шаг 3. Необходимо настроить маршрутизатор.

a.	Необходимо подключиться к маршрутизатору с помощью консоли и активировать привилегированный режим EXEC.

b.	Необходимо войти в режим конфигурации.

c.	Необходимо отключить поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

d.	Необходимо назначить class в качестве зашифрованного пароля привилегированного режима EXEC.

e.	Необходимо назначить cisco в качестве пароля консоли и включите вход в систему по паролю.

f.	Необходимо назначить cisco в качестве пароля VTY и включите вход в систему по паролю.

g.	Необходимо зашифровать открытые пароли.

h.	Необходимо создать баннер, который предупреждает о запрете несанкционированного доступа.

i.	Необходимо настроить и активировать на маршрутизаторе интерфейс G0/0/1, используя информацию, приведенную в таблице адресации.

j.	Необходимо созранить текущую конфигурацию в файл загрузочной конфигурации.

Ниже привожу отображение заданных настроек с помощью команды `running-config`:

```
R1#show running
Building configuration...

Current configuration : 797 bytes
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
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
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
ATTENTION!!! Unauthorized acces is strictly proibited! ^C
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
```

Эти настройки были скопированы в `startup-config` с помощью соответствующей команды.

#### Шаг 4. Необходимо настроить компьютер PC-A.

a.	Необходимо настроить для PC-A IP-адрес и маску подсети.

b.	Необходимо настроить для PC-A шлюз по умолчанию.

Ниже привожу отображение настроек PC-A:

```
C:\>ipconfig

FastEthernet0 Connection:(default port)

   Connection-specific DNS Suffix..: 
   Link-local IPv6 Address.........: FE80::206:2AFF:FE3A:623
   IPv6 Address....................: ::
   IPv4 Address....................: 192.168.1.3
   Subnet Mask.....................: 255.255.255.0
   Default Gateway.................: ::
                                     192.168.1.1
```
#### Шаг 5. Необходимо проверить подключение к сети.

Необходимо отправить с PC-A команду Ping на маршрутизатор R1. Если эхо-запрос с помощью команды ping не проходит, тогда необходимо найти и устранить неполадки подключения.

После можно закрыть окно настройки.

Ниже привожу результат эхо-запроса в адрес коммутатора R1:

```
C:\>ping 192.168.1.1

Pinging 192.168.1.1 with 32 bytes of data:

Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255

Ping statistics for 192.168.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
```

### Часть 2. Настройка маршрутизатора для доступа по протоколу SSH

Подключение к сетевым устройствам по протоколу Telnet сопряжено с риском для безопасности, поскольку вся информация передается в виде открытого текста. Протокол SSH шифрует данные сеанса и обеспечивает аутентификацию устройств, поэтому для удаленных подключений рекомендуется использовать именно этот протокол. В части 2 вам нужно настроить маршрутизатор для приема соединений SSH по линиям VTY

#### Шаг 1. Настройте аутентификацию устройств.

При генерации ключа шифрования в качестве его части используются имя устройства и домен. Поэтому эти имена необходимо указать перед вводом команды `crypto key`.

Необходимо открыть окно конфигурации, далее:

a.	Задать имя устройства.

Имя устройства было задано мной на этапе первичной настройки маршрутизатора

b.	Задать домен для устройства.

Домен задаю с помощью команды `ip-domain name`, в качестве имени домена задаю `otus.ru`

#### Шаг 2. Необходимо создать ключ шифрования с указанием его длины.

Ключ задаю с помощью команды `crypto key generate rs general-keys 1024`, где числом 1024 задаю размер шифрования ключа.

```
R1(config)#crypto key generate rsa general-keys modulus 1024
The name for the keys will be: R1.otus.ru

% The key modulus size is 1024 bits
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]
*Mar 1 0:20:54.114: %SSH-5-ENABLED: SSH 1.99 has been enabled
```

Задаю версию SSH 2:

```
R1(config)#
R1(config)#ip ssh v
R1(config)#ip ssh version 2
R1(config)#
R1(config)#end
R1#
%SYS-5-CONFIG_I: Configured from console by console

R1#show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
R1#
```

#### Шаг 3. Необходимо создать имя пользователя в локальной базе учетных записей.

Необходимо настроить имя пользователя, используя `admin` в качестве имени пользователя и `Adm1nP@55` в качестве пароля.

`R1(config)#username admin privilege 15 secret Adm1nP@55`


#### Шаг 4. Необходимо активировать протокол SSH на линиях VTY.

a.	Необходимо активировать протоколы Telnet и SSH на входящих линиях VTY с помощью команды `transport input`.

```
R1(config-line)#transport input telnet
R1(config-line)#transport input ssh
```

b.	Необходимо изменить способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.

Выполняю с помощью команды login local:
`R1(config-line)#login local`

#### Шаг 5. Необходимо сохранить текущую конфигурацию в файл загрузочной конфигурации.
```
R1#copy
R1#copy run
R1#copy running-config star
Destination filename [startup-config]? 
Building configuration...
[OK]
R1#
R1#
R1#
R1#show startup-c
Using 1118 bytes
!
version 15.4
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
security passwords min-length 8
!
hostname R1
!
login block-for 120 attempts 3 within 60
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
username admin privilege 15 secret 5 $1$mERr$qJb.eHvBN7S590aq.dpRL.
!
!
!
!
!
!
!
!
no ip domain-lookup
ip domain-name otus.ru
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
 ip address 192.168.1.1 255.255.255.0
 duplex auto
 speed auto
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
ip access-list extended sl_def_acl
 deny tcp any any eq telnet
 deny tcp any any eq www
 deny tcp any any eq 22
 permit tcp any any eq 22
!
banner motd ^C
ATTENTION!!! Unauthorized acces is strictly proibited! ^C
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
 login local
 transport input ssh
!
!
!
end
```


#### Шаг 6. Необходимо установить соединение с маршрутизатором по протоколу SSH.

a.	Для этого запустить Tera Term с PC-A.

b.	После установить SSH-подключение к R1. Use the username admin and password Adm1nP@55. В результате должно получиться установить SSH-подключение к R1.


Подключение выполняю с помощью следующей команды:

```
C:\>ssh -l admin 192.168.1.1

Password: 


ATTENTION!!! Unauthorized acces is strictly proibited! 

R1#
```

Видно, что маршрутизатор запрашивает пароль, после ввода корректного отображает баннер MOTD и переходит в привилегированный режим.

### Часть 3. Настройка коммутатора для доступа по протоколу SSH

В части 3 предстоит настроить коммутатор для приема подключений по протоколу SSH, а затем установить SSH-подключение с помощью программы Tera Term.

#### Шаг 1. Необходимо настроить основные параметры коммутатора.

Необходимо открыть окно конфигурации

a.	Необходимо подключиться к коммутатору с помощью консольного подключения и активировать привилегированный режим EXEC.

b.	Необходимо войти в режим конфигурации.

c.	Необходимо отключить поиск DNS, чтобы предотвратить попытки маршрутизатора неверно преобразовывать введенные команды таким образом, как будто они являются именами узлов.

d.	Необходимо назначить class в качестве зашифрованного пароля привилегированного режима EXEC.

e.	Необходимо назначить cisco в качестве пароля консоли и включите вход в систему по паролю.

f.	Необходимо назначить cisco в качестве пароля VTY и включите вход в систему по паролю.

g.	Необходимо зашифорвать открытые пароли.

h.	Необходимо создать баннер, который предупреждает о запрете несанкционированного доступа.

i.	Необходимо настроить и активировать на коммутаторе интерфейс VLAN 1, используя информацию, приведенную в таблице адресации.

j.	Необходимо сохранить текущую конфигурацию в файл загрузочной конфигурации.

Ниже привожу результат настройки:

```
S1#show startup-config 
Building configuration...

Current configuration : 1331 bytes
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
 ip address 192.168.1.11 255.255.255.0
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

#### Шаг 2. Необходимо настроить коммутатор для соединения по протоколу SSH.

Для настройки протокола SSH на коммутаторе необходимо использовать те же команды, которые применялись для аналогичной настройки маршрутизатора в части 2.

a.	Необходимо настроить имя устройства, как указано в таблице адресации.

Имя задано при первичной настройке

b.	Необходимо задать домен для устройства.

Задаю следующим образом:
`S1(config)#ip domain otus.ru`

c.	Необходимо создать ключ шифрования с указанием его длины.

Задаю так же, как и для маршрутизатора:
```
S1(config)#crypto key generate rsa general-keys modulus 1204
The name for the keys will be: S1.otus.ru

% The key modulus size is 1204 bits
% Generating 1204 bit RSA keys, keys will be non-exportable...[OK]
*Mar 1 2:30:11.755: %SSH-5-ENABLED: SSH 1.99 has been enabled
```

d.	Необходимо создать имя пользователя в локальной базе учетных записей.

Задаю командой:

`S1(config)#username admin privilege 15 secret Adm1nP@55`

e.	Необходимо активировать протоколы Telnet и SSH на линиях VTY.

Активирую так же, как и для маршрутизатора:
```
S1(config)#line vty 0 4
S1(config-line)#
S1(config-line)#
S1(config-line)#transport input ssh
S1(config-line)#
S1(config-line)#
S1(config-line)#login local
S1(config-line)#
S1(config-line)#
S1(config-line)#line vty 5 15
S1(config-line)#
S1(config-line)#
S1(config-line)#
S1(config-line)#transport input ssh
S1(config-line)#
S1(config-line)#login local
```

Указываю версию SSH:

```
S1(config)#ip ssh version 2
S1(config)#
S1(config)#
S1(config)#
S1(config)#end
S1#
%SYS-5-CONFIG_I: Configured from console by console

S1#show ip ssh
SSH Enabled - version 2.0
Authentication timeout: 120 secs; Authentication retries: 3
```


f.	Необходимо изменить способ входа в систему таким образом, чтобы использовалась проверка пользователей по локальной базе учетных записей.

Команда `login local` применена в предыдущем пункте

#### Шаг 3. Необходимо установить соединение с коммутатором по протоколу SSH.

Необходимо запустить программу Tera Term на PC-A, затем установить подключение по протоколу SSH к интерфейсу SVI коммутатора S1.

```
C:\>ssh -l admin 192.168.1.11

Password: 
% Login invalid


Password: 


ATTENTION!!! Unauthorized acces is strictly prohibited! 

S1#
```


**Вопрос:** 
Удалось ли вам установить SSH-соединение с коммутатором?

**Ответ:**
В приведенном выше наборе команд и сообщений виден результат: при вводе некорректного пароля маршрутизатор отказал в аутентификации, при вводе корректного - пропустил в привилегированный режим.

### Часть 4. Настройка протокола SSH с использованием интерфейса командной строки (CLI) коммутатора

Клиент SSH встроен в операционную систему Cisco IOS и может запускаться из интерфейса командной строки. В части 4 предстоит установить соединение с маршрутизатором по протоколу SSH, используя интерфейс командной строки коммутатора.

#### Шаг 1. Необходимо посмотреть доступные параметры для клиента SSH в Cisco IOS.

Для этого необходимо открыть окно конфигурации

И далее необходимо использовать вопросительный знак (?), чтобы отобразить варианты параметров для команды ssh.

```
S1# ssh? 
  -c Select encryption algorithm
  -l Log in using this user name
  -m Select HMAC algorithm
  -o Specify options
  -p Connect to this port
  -v Specify SSH Protocol Version
  -vrf Specify vrf name
  WORD IP-адрес или имя хоста удаленной системы
  ```

#### Шаг 2. Необходимо установить с коммутатора S1 соединение с маршрутизатором R1 по протоколу SSH.

a.	Чтобы подключиться к маршрутизатору R1 по протоколу SSH, необходимо ввести команду `–l admin`. Это позволит войти в систему под именем admin. При появлении приглашения необходимоввести в качестве пароля `Adm1nP@55`

```
S1# ssh -l admin 192.168.1.1
Password: 
Authorized Users Only!
R1>
```
b.	Чтобы вернуться к коммутатору S1, не закрывая сеанс SSH с маршрутизатором R1, необходимо нажать комбинацию клавиш `Ctrl+Shift+6`. После отпустить клавиши `Ctrl+Shift+6` и нажать `x`. В результате отобразится приглашение привилегированного режима EXEC коммутатора.

```
R1>
S1#
```

c.	Чтобы вернуться к сеансу SSH на R1, необходимо нажать клавишу Enter в пустой строке интерфейса командной строки. Чтобы увидеть окно командной строки маршрутизатора, необходимо нажать клавишу Enter еще раз.

```
S1#
[Resuming connection 1 to 192.168.1.1 ... ]

R1>
```

d.	Чтобы завершить сеанс SSH на маршрутизаторе R1, необходимо ввести в командной строке маршрутизатора команду `exit`.

```
R1# exit

[Connection to 192.168.1.1 closed by foreign host]
S1#
```

Привожу полную и неразрывную последовательность команд:

```
S1#
S1#
S1#
S1#ssh -l admin 192.168.1.1

Password: 
% Login invalid


Password: 


ATTENTION!!! Unauthorized acces is strictly proibited! 

R1#
R1#
R1#
R1#
S1#
S1#
[Resuming connection 1 to 192.168.1.1 ... ]

R1#
R1#
R1#exit

[Connection to 192.168.1.1 closed by foreign host]
S1#
```


**Вопрос:**
Какие версии протокола SSH поддерживаются при использовании интерфейса командной строки?

**Ответ:**
```
S1#ssh -v ?
  1  Protocol Version 1
  2  Protocol Version 2
```

Теперь можно закрыть окно настройки.

**Вопрос для повторения**
Как предоставить доступ к сетевому устройству нескольким пользователям, у каждого из которых есть собственное имя пользователя?

**Ответ:**
1. Необходимо настроить в локальной базе учетных записей сетевого устройства имена и пароли для каждого пользователя или для групп пользователей 
2. Активировать SSH для каждого имени на сетевом устройстве
3. Указать версию SSH
4. Включить проверку пользователей по локальной базе учетных записей