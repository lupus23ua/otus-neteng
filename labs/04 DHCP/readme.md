## [Задание 1](7.4.2%20Lab%20-%20Implement%20DHCPv4.docx): Настроить DHCPv4.

### Топология в EVE-NG:

![](topology.png)

### Таблица ip-адресов:
|Device|Interface|IP Address|Subnet Mask|Default Gateway|
|:-|:-|:-:|:-:|:-:|
|R1|e0/0|10.0.0.1|255.255.255.252|N/A|
||e0/1||||
||e0/1.100|192.168.1.1|255.255.255.192|N/A|
||e0/1.200|192.168.1.65|255.255.255.224|N/A|
||e0/1.1000||||
|R2|e0/0|10.0.0.2|255.255.255.252|N/A|
||e0/1|192.168.1.97|255.255.255.240|N/A|
|S1|VLAN 200|192.168.1.66|255.255.255.192|192.168.1.65|
|S2|VLAN 1|192.168.1.98|255.255.255.240|192.168.1.97|
|VPC1|DHCP|DHCP|DHCP|
|VPC2|DHCP|DHCP|DHCP|

### Таблица VLAN:
| VLAN	| Name	|Interface Assigned|
|:-|:-:|:-|
|1|N/A|S2: e0/0-3|
|100|Clients|S1: e0/3|
|200|Management|S1: VLAN 200|
|999|Parking_Lot|S1: e0/1-2|
|1000|Native|N/A|

## Часть 1: Создание сети и настройка основных параметров устройства

Subnet the network 192.168.1.0/24 to meet the following requirements:
1.	One subnet, “Subnet A”, supporting 58 hosts (the client VLAN at R1).  
Subnet A: 192.168.1.0/26  
Record the first IP address in the Addressing Table for R1 e0/1.100.
2.	One subnet, “Subnet B”, supporting 28 hosts (the management VLAN at R1).  
Subnet B: 192.168.1.64/27  
Record the first IP address in the Addressing Table for R1 e0/1.200. Record the second IP address in the Address Table for S1 VLAN 200 and enter the associated default gateway.
3.	One subnet, “Subnet C”, supporting 12 hosts (the client network at R2).  
Subnet C: 192.168.1.96/28  
Record the first IP address in the Addressing Table for R2 e0/1.

### R1-R2 basic settings:
```
enable
configure terminal
hostname R1
no ip domain-lookup
enable secret class
line console 0
exec-timeout 0 0
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
interface ethernet 0/0
ip address 10.0.0.1 255.255.255.252
no shutdown
end
clock set 17:30:00 01 APRIL 2022
write
```
```
enable
configure terminal
hostname R2
no ip domain-lookup
enable secret class
line console 0
exec-timeout 0 0
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
interface ethernet 0/0
ip address 10.0.0.2 255.255.255.252
no shutdown
end
clock set 17:30:00 01 APRIL 2022
write
```
### R1 inter-VLAN routing settings and default route:
```
interface ethernet 0/1
no shutdown
interface ethernet 0/1.100
encapsulation dot1Q 100
ip address 192.168.1.1 255.255.255.192
description Clients
interface ethernet 0/1.200
encapsulation dot1Q 200
ip address 192.168.1.65 255.255.255.224
description Management
interface ethernet 0/1.1000
encapsulation dot1Q 1000 native
description Native
ip route 0.0.0.0 0.0.0.0 10.0.0.2
end
write
```
### R2 routing settings:
```
interface ethernet 0/1
ip address 192.168.1.97 255.255.255.240
no shutdown
ip route 0.0.0.0 0.0.0.0 10.0.0.1
end
write
```
```
R2#ping 192.168.1.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```
### S1-S2 basic settings:
```
enable
configure terminal
hostname S1
no ip domain-lookup
enable secret class
line console 0
exec-timeout 0 0
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
end
clock set 17:52:00 01 APRIL 2022
write
```
### S1-S2 VLAN and routing settings:
####S1
```
vlan 100
name Clients
vlan 200
name Management
vlan 999
name Parking_Lot
vlan 1000
name Native
interface Vlan200
ip address 192.168.1.66 255.255.255.192
no shutdown
ip route 0.0.0.0 0.0.0.0 192.168.1.65
interface ethernet 0/0
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk allowed vlan 100,200,999,1000
switchport trunk native vlan 1000
interface Ethernet0/3
switchport mode access
switchport access vlan 100
spanning-tree portfast
interface range ethernet 0/1 - 2
switchport mode access
switchport access vlan 999
shutdown
```
####S2
```
interface vlan 1
ip address 192.168.1.98 255.255.255.240
no shutdown
interface range ethernet 0/1 - 2
shutdown
ip route 0.0.0.0 0.0.0.0 192.168.1.97
```


## [Задание 2](8.5.1%20Lab%20-%20Configure%20DHCPv6.docx): Настроить DHCPv6.
