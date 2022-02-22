## Задание: настроить "роутер на палочке" и Inter-VLAN маршрутизацию.

Топология в EVE-NG:

![](https://github.com/lupus23ua/otus-neteng/blob/main/labs/02%20VLAN/LAB02-EVE-TOPO.png)

Таблица ip-адресов:

![](https://github.com/lupus23ua/otus-neteng/blob/main/labs/02%20VLAN/LAB02-ADDRESS-TABLE.png)

Команды для настройки:
```
R1:
Router(config)#interface ethernet 0/0.3
Router(config-subif)#description GW VLAN 3
Router(config-subif)#encapsulation dot1Q 3
Router(config-subif)#ip add 192.168.3.1 255.255.255.0
Router(config-subif)#exit
Router(config)#interface ethernet 0/0.4        
Router(config-subif)#description GW VLAN 4
Router(config-subif)#encapsulation dot1Q 4
Router(config-subif)#ip add 192.168.4.1 255.255.255.0
Router(config-subif)#exit
Router(config)#interface ethernet 0/0.8
Router(config-subif)#exit
```

S1
