# [Задание](https://github.com/lupus23ua/otus-neteng/blob/main/labs/03%20STP/3_1_2_12_Lab_Building_a_Switched_Network_with_Redundant_Links.docx): развертывание коммутируемой сети с резервными каналами.

### Топология в EVE-NG:

![](https://github.com/lupus23ua/otus-neteng/blob/main/labs/03%20STP/topology.png)

### Таблица ip-адресов:

![](https://github.com/lupus23ua/otus-neteng/blob/main/labs/03%20STP/address%20table.png)

## [Экспорт лабораторной в eve-ng](https://github.com/lupus23ua/otus-neteng/blob/main/labs/03%20STP/Lab-03_STP-eve-ng.zip)

## Часть 1: Создание сети и настройка основных параметров устройства
### SW1-SW3 Basic settings:
```
enable
configure terminal
hostname SW1
no ip domain-lookup
enable secret class
line console 0
password cisco
login
logging synchronous
line vty 0 4
password cisco
login
logging synchronous
service password-encryption
banner motd ^
##########################################
#                                        #
#        Authorised Access Only          #
#                                        #
##########################################
^
interface vlan 1
ip address 192.168.1.1 255.255.255.0
no shutdown
end
write
```

### Проверка связи между коммутаторами:
```
SW1#ping 192.168.1.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/2 ms

SW1#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
```
```
SW2#ping 192.168.1.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.3, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 2/2/3 ms
```

## Часть 2: Определение корневого моста
### SW1-SW3 configuration:
```
interface range ethernet 0/0 - 3
shutdown
switchport trunk encapsulation dot1q
switchport mode trunk
interface range ethernet 0/1 , ethernet 0/3
no shutdown
end
```
```
SW1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg FWD 100       128.2    P2p
Et0/3               Desg FWD 100       128.4    P2p
```
```
SW2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    P2p
Et0/3               Desg FWD 100       128.4    P2p
```
```
SW3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Altn BLK 100       128.2    P2p
Et0/3               Root FWD 100       128.4    P2p
```

### Состояние портов:

![](https://github.com/lupus23ua/otus-neteng/blob/main/labs/03%20STP/port%20states.png)

### Ответы на вопросы:

- Q: Какой коммутатор является корневым мостом?
    - A: SW1.

- Q: Почему этот коммутатор был выбран протоколом spanning-tree в качестве корневого моста?
    - A: Одинкаовый Priority и наименьший MAC-адрес.


- Q: Какие порты на коммутаторе являются корневыми портами?
    - A: Порты на некорневых коммутаторах в построенном дереве STP, направленные в сторону корневого коммутатора.


- Q: Какие порты на коммутаторе являются назначенными портами?
    - A: Порты на всех коммутаторах в построенном дереве STP, направленные не на корневой коммутатор и не зарезервированные.


- Q: Какой порт отображается в качестве альтернативного и в настоящее время заблокирован?
    - A: SW3 e0/1.


- Q: Почему протокол spanning-tree выбрал этот порт в качестве невыделенного (заблокированного) порта?
    - A: Потому что у SW3 самый высокий идентификатор BID и e0/3 designated path cost 0, а у e0/1 designated path cost 100 (параметры path cost взяты из консоли, в теории должно быть 100 и 200)

## Часть 3: Наблюдение за процессом выбора протоколом STP порта, исходя из стоимости портов
```
SW3(config)#interface ethernet 0/3
SW3(config-if)#spanning-tree cost 90
SW3(config-if)#do show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        90
             Port        4 (Ethernet0/3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Desg LIS 100       128.2    P2p
Et0/3               Root FWD 90        128.4    P2p
```
```
SW2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        2 (Ethernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/1               Root FWD 100       128.2    P2p
Et0/3               Altn BLK 100       128.4    P2p
```
- Q: Почему протокол spanning-tree заменяет ранее заблокированный порт на назначенный порт и блокирует порт, который был назначенным портом на другом коммутаторе?
    - A: После изменения настроек дерево STP перестраивается заново.

## Часть 4: Наблюдение за процессом выбора протоколом STP порта, исходя из приоритета портов

### Конфигурация SW1-SW3:
```
interface range ethernet 0/0 , ethernet 0/2
no shutdown
```
```
SW1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0100
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Desg FWD 100       128.1    P2p
Et0/1               Desg FWD 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
Et0/3               Desg FWD 100       128.4    P2p
```
```
SW2#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        1 (Ethernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0200
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Root FWD 100       128.1    P2p
Et0/1               Altn BLK 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
Et0/3               Desg FWD 100       128.4    P2p
```
```
SW3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     aabb.cc00.0100
             Cost        100
             Port        3 (Ethernet0/2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     aabb.cc00.0300
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  15  sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Et0/0               Altn BLK 100       128.1    P2p
Et0/1               Altn BLK 100       128.2    P2p
Et0/2               Root FWD 100       128.3    P2p
Et0/3               Altn BLK 100       128.4    P2p
```
- Q: Какой порт выбран протоколом STP в качестве порта корневого моста на каждом коммутаторе некорневого моста?
    - A: SW2 e0/0, SW3 e0/2.

- Q: Почему протокол STP выбрал эти порты в качестве портов корневого моста на этих коммутаторах?
    - A: На SW2 у портов e0/0 и e0/1 одинаковые Priority и Path Cost, но у порта e0/0 меньший номер. На SW3 то же самое с портами e0/2 и e0/3.

#### Вопросы для повторения

- Q: Какое значение протокол STP использует первым после выбора корневого моста, чтобы определить выбор порта?
    - A: Path Cost

- Q: Если первое значение на двух портах одинаково, какое следующее значение будет использовать протокол STP при выборе порта?
    - A: Port Priority

- Q: Если оба значения на двух портах равны, каким будет следующее значение, которое использует протокол STP при выборе порта?
    - A: Port Number
