## [Задание 1](https://github.com/lupus23ua/otus-neteng/blob/main/labs/04%20DHCP/7.4.2%20Lab%20-%20Implement%20DHCPv4.docx): Настроить DHCPv4.

###### Топология в EVE-NG:

![](https://github.com/lupus23ua/otus-neteng/blob/main/labs/03%20STP/topology.png)

### Таблица ip-адресов:
| Device	| Interface	| IP Address	| Subnet Mask	| Default Gateway |
|:-|:-|:-:|:-:|:-:|
|R1|G0/0/0|10.0.0.1|255.255.255.252|N/A|
||G0/0/1||||
||G0/0/1.100||||
||G0/0/1.200||||
||G0/0/1.1000||||
|R2|G0/0/0|10.0.0.2|255.255.255.252|N/A|
||G0/0/1|||
|S1|VLAN 200||||
|S2|VLAN 1||||
|PC-A|NIC|DHCP|DHCP|DHCP|
|PC-B|NIC|DHCP|DHCP|DHCP|

### Таблица VLAN:
| VLAN	| Name	|Interface Assigned|
|:-|:-:|:-|
|1|N/A|S2: F0/18|
|100|Clients|S1: F0/6|
|200|Management|S1: VLAN 200 |
|999|Parking_Lot|S1: F0/1-4, F0/7-24, G0/1-2|
|1000|Native|N/A|


## [Задание 2](https://github.com/lupus23ua/otus-neteng/blob/main/labs/04%20DHCP/8.5.1%20Lab%20-%20Configure%20DHCPv6.docx): Настроить DHCPv6.
