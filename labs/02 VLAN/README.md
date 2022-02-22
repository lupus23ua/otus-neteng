## Задание: настроить "роутер на палочке" и Inter-VLAN маршрутизацию.

Топология в EVE-NG:

![](https://github.com/lupus23ua/otus-neteng/blob/main/labs/02%20VLAN/LAB02-EVE-TOPO.png)

Таблица ip-адресов:

![](https://github.com/lupus23ua/otus-neteng/blob/main/labs/02%20VLAN/LAB02-ADDRESS-TABLE.png)

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

R1(config)#interface ethernet 0/0.3
R1(config-subif)#description GW VLAN 3
R1(config-subif)#encapsulation dot1Q 3
R1(config-subif)#ip add 192.168.3.1 255.255.255.0
R1(config-subif)#exit
R1(config)#interface ethernet 0/0.4        
R1(config-subif)#description GW VLAN 4
R1(config-subif)#encapsulation dot1Q 4
R1(config-subif)#ip add 192.168.4.1 255.255.255.0
R1(config-subif)#exit
R1(config)#interface ethernet 0/0.8
R1(config-subif)#exit
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
