## Задание: настроить "роутер на палочке" и Inter-VLAN маршрутизацию.

Топология в EVE-NG:

![](labs/02 VLAN/LAB02-EVE-TOPO.png)

Таблица ip-адресов:

![](labs/02 VLAN/LAB02-ADDRESS-TABLE.png)

Таблица VLAN:

![](labs/02 VLAN/LAB02-VLAN-TABLE.png)

Команды для настройки:
```
R1 Basic Settings:
Router>enable
Router#configure terminal
Router(config)#hostname R1
R1(config)#no ip domain-lookup
R1(config)#enable secret class
R1(config)#line console 0
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#line vty 0 4
R1(config-line)#password cisco
R1(config-line)#login
R1(config-line)#service password-encryption
R1(config)#banner motd ^
Enter TEXT message.  End with the character '^'.
##########################################
#                                        #
#        Authorised Access Only          #
#                                        #
##########################################
^
R1(config)#end
R1#clock set 18:35:00 22 FEBRUARY 2022
R1#write
```
```
S1 and S2 Basic Settings:
Switch>enable
Switch#configure terminal
Switch(config)#hostname S1
S1(config)#no ip domain-lookup
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#line vty 0 4
S1(config-line)#password cisco
S1(config-line)#login
S1(config-line)#service password-encryption
S1(config)#banner motd ^
Enter TEXT message.  End with the character '^'.
##########################################
#                                        #
#        Authorised Access Only          #
#                                        #
##########################################
^
S1(config)#end
S1#clock set 18:35:00 22 FEBRUARY 2022
S1#write
```
```
PC-A config:
ip 192.168.3.3 255.255.255.0 192.168.3.1
```
```
PC-B config:
ip 192.168.4.3 255.255.255.0 192.168.4.1
```
```
S1 VLAN configuration:
S1#configure terminal
S1(config)#vlan 3
S1(config-vlan)#name Management
S1(config-vlan)#vlan 4
S1(config-vlan)#name Operations
S1(config-vlan)#vlan 7
S1(config-vlan)#name ParkingLot
S1(config-vlan)#vlan 8
S1(config-vlan)#name Native
S1(config-vlan)#interface vlan 3
S1(config-if)#ip address 192.168.3.11 255.255.255.0
S1(config-if)#no shutdown
S1(config-if)#ip route 0.0.0.0 0.0.0.0 192.168.3.1
S1(config)#interface ethernet 0/0
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 7
S1(config-if)#interface ethernet 0/1
S1(config-if)#switchport mode access
S1(config-if)#switchport access vlan 3
S1(config-if)#interface range ethernet 0/2 - 3
S1(config-if-range)#switchport trunk encapsulation dot1q
S1(config-if-range)#switchport mode trunk
S1(config-if-range)#switchport trunk allowed vlan 3,4,8
S1(config-if-range)#switchport trunk native vlan 8
S1(config-if-range)#end
S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active
3    Management                       active    Et0/1
4    Operations                       active
7    ParkingLot                       active    Et0/0
8    Native                           active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
S1#write
```
```
S2 VLAN configuration:
S2#configure terminal
S2(config)#vlan 3
S2(config-vlan)#name Management
S2(config-vlan)#vlan 4
S2(config-vlan)#name Operations
S2(config-vlan)#vlan 7
S2(config-vlan)#name ParkingLot
S2(config-vlan)#vlan 8
S2(config-vlan)#name Native
S2(config-vlan)#interface vlan 3
S2(config-if)#ip address 192.168.3.12 255.255.255.0
S2(config-if)#no shutdown
S2(config-if)#ip route 0.0.0.0 0.0.0.0 192.168.3.1
S2(config)#interface ethernet 0/0
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 7
S2(config-if)#interface ethernet 0/2
S2(config-if)#switchport mode access
S2(config-if)#switchport access vlan 4
S2(config-if)#interface ethernet 0/3
S2(config-if)#switchport trunk encapsulation dot1q
S2(config-if)#switchport mode trunk
S2(config-if)#switchport trunk allowed vlan 3,4,8
S2(config-if)#switchport trunk native vlan 8
S2(config-if)#end
S2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/1
3    Management                       active
4    Operations                       active    Et0/2
7    ParkingLot                       active    Et0/0
8    Native                           active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
S2#write
```
```
R1 Routing configuration:
R1#configure terminal
R1(config)#interface ethernet 0/3
R1(config-if)#no shutdown
R1(config-if)#interface ethernet 0/0.3
R1(config-subif)#description GW VLAN 3
R1(config-subif)#encapsulation dot1Q 3
R1(config-subif)#ip add 192.168.3.1 255.255.255.0
R1(config-subif)#interface ethernet 0/0.4        
R1(config-subif)#description GW VLAN 4
R1(config-subif)#encapsulation dot1Q 4
R1(config-subif)#ip add 192.168.4.1 255.255.255.0
R1(config-subif)#interface ethernet 0/0.8
R1(config-subif)#end
R1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                unassigned      YES unset  up                    up
Ethernet0/0.3              192.168.3.1     YES manual up                    up
Ethernet0/0.4              192.168.4.1     YES manual up                    up
Ethernet0/0.8              unassigned      YES unset  up                    up
Ethernet0/1                unassigned      YES unset  administratively down down
Ethernet0/2                unassigned      YES unset  administratively down down
Ethernet0/3                unassigned      YES unset  down                  down
R1#write
```
```
Testing PC-A connection:
VPCS> show ip
NAME        : VPCS[1]
IP/MASK     : 192.168.3.3/24
GATEWAY     : 192.168.3.1
DNS         :
MAC         : 00:50:79:66:68:04
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.3.1
84 bytes from 192.168.3.1 icmp_seq=1 ttl=255 time=2.575 ms
84 bytes from 192.168.3.1 icmp_seq=2 ttl=255 time=1.765 ms

VPCS> ping 192.168.4.3
84 bytes from 192.168.4.3 icmp_seq=1 ttl=63 time=2.191 ms
84 bytes from 192.168.4.3 icmp_seq=2 ttl=63 time=3.418 ms

VPCS> trace 192.168.4.3 -P 1
trace to 192.168.4.3, 8 hops max (ICMP), press Ctrl+C to stop
 1   192.168.3.1   1.934 ms  1.059 ms  1.369 ms
 2   192.168.4.3   2.651 ms  1.775 ms  1.319 ms


```
```
Testing PC-B connection:
VPCS> show ip
NAME        : VPCS[1]
IP/MASK     : 192.168.4.3/24
GATEWAY     : 192.168.4.1
DNS         :
MAC         : 00:50:79:66:68:05
LPORT       : 20000
RHOST:PORT  : 127.0.0.1:30000
MTU         : 1500

VPCS> ping 192.168.4.1
84 bytes from 192.168.4.1 icmp_seq=1 ttl=255 time=3.577 ms
84 bytes from 192.168.4.1 icmp_seq=2 ttl=255 time=2.645 ms

VPCS> ping 192.168.3.3
84 bytes from 192.168.3.3 icmp_seq=1 ttl=63 time=3.990 ms
84 bytes from 192.168.3.3 icmp_seq=2 ttl=63 time=4.370 ms

VPCS> trace 192.168.3.3 -P 1
trace to 192.168.3.3, 8 hops max (ICMP), press Ctrl+C to stop
 1   192.168.4.1   2.929 ms  2.179 ms  2.015 ms
 2   192.168.3.3   3.121 ms  3.417 ms  3.041 ms
```
