# Лабораторная работа №1. Проектирование адресного пространства
Задачи:
1. Собрать топологию CLOS
2. Распределить адресное пространство для Underlay сети
3. Зафиксировать в документации план работ, адресное пространство, схему сети, настройки
### Схема сети
Данная схема собрана в PNETLab. Топология состоит из двух SPINEs и трёх LEAFs. Каждый LEAF подключается двумя линками к SPINE. Также представлено клиентское оборудование (четыре штуки), каждое из которых подключается одним линком к LEAF.
![](lab1.jpg)
### Список устройств
|Device|Role|Model|
|---|---|---|
vos-spine-sw01|spine|Cisco NX-OSv 9K
vos-spine-sw02|spine|Cisco NX-OSv 9K
vos-leaf-sw01|leaf|Cisco NX-OSv 9K
vos-leaf-sw02|leaf|Cisco NX-OSv 9K
vos-leaf-sw03|leaf|Cisco NX-OSv 9K
Client_red|client device|Virtual PC
Client_yellow|client device|Virtual PC
Client_black|client device|Virtual PC
Client_white|client device |Virtual PC
### План адресации
Для всей площадки будет использоваться сеть **10.0.0.0/16**, которая будет дробиться на более мелкие сети в зависимости от цели использования.
На данный момент план адресации выглядит так:
|Network|||Description|
|---|---|---|---|
10.0.0.0/24|||EVPN Fabric|
||10.0.0.0/25||P2P for Spine-Leaf
|||10.0.0.0/31|vos-spine-sw01 <--> vos-leaf-sw01
|||10.0.0.2/31|vos-spine-sw02 <--> vos-leaf-sw01
|||10.0.0.4/31|vos-spine-sw01 <--> vos-leaf-sw02
|||10.0.0.6/31|vos-spine-sw02 <--> vos-leaf-sw02
|||10.0.0.8/31|vos-spine-sw01 <--> vos-leaf-sw03
|||10.0.0.10/31|vos-spine-sw02 <--> vos-leaf-sw03
||10.0.0.128/25||Loopback addresses for Spine, Leaf
|||10.0.0.128/32| loopback vos-spine-sw01
|||10.0.0.129/32| loopback vos-spine-sw02
|||10.0.0.130/32| loopback vos-leaf-sw01
|||10.0.0.131/32| loopback vos-leaf-sw02
|||10.0.0.132/32| loopback vos-leaf-sw03
### Underlay network
Для стыка между SPINE-LEAF выделены /31 P2P сети. Где первый адрес в сети - адрес для SPINE. Второй адрес - адрес для LEAF
+ **10.0.0.0/31 vos-spine-sw01 <--> vos-leaf-sw01**
    + 10.0.0.0 vos-spine-sw01
    + 10.0.0.1 vos-leaf-sw01
+ **10.0.0.2/31 vos-spine-sw02 <--> vos-leaf-sw01**
    + 10.0.0.2 vos-spine-sw02
    + 10.0.0.3 vos-leaf-sw01
+ **10.0.0.4/31 vos-spine-sw01 <--> vos-leaf-sw02**
    + 10.0.0.4 vos-spine-sw01
    + 10.0.0.5 vos-leaf-sw02
+ **10.0.0.6/31 vos-spine-sw02 <--> vos-leaf-sw02**
    + 10.0.0.6 vos-spine-sw02
    + 10.0.0.7 vos-leaf-sw02
+ **10.0.0.8/31 vos-spine-sw01 <--> vos-leaf-sw03**
    + 10.0.0.8 vos-spine-sw01
    + 10.0.0.9 vos-leaf-sw03
+ **10.0.0.10/31 vos-spine-sw02 <--> vos-leaf-sw03**
    + 10.0.0.10 vos-spine-sw02
    + 10.0.0.11 vos-leaf-sw03
### Конфигурация
В рамках данной лабораторной работы были настроены линки между каждой парой SPINE - LEAF (description, ip address, no ip redirects, mtu)
##### Конфигурация vos-spine-sw01
```
interface Ethernet1/1
  description vos-leaf-sw01 [Eth1/1]
  mtu 9216
  no ip redirects
  ip address 10.0.0.0/31
  no shutdown

interface Ethernet1/2
  description vos-leaf-sw02 [Eth1/1]
  mtu 9216
  no ip redirects
  ip address 10.0.0.4/31
  no shutdown

interface Ethernet1/3
  description vos-leaf-sw03 [Eth1/1]
  mtu 9216
  no ip redirects
  ip address 10.0.0.8/31
  no shutdown
```
##### Конфигурация vos-spine-sw02
```interface Ethernet1/1
  description vos-leaf-sw01 [Eth1/2]
  mtu 9216
  no ip redirects
  ip address 10.0.0.2/31
  no shutdown

interface Ethernet1/2
  description vos-leaf-sw02 [Eth1/2]
  mtu 9216
  no ip redirects
  ip address 10.0.0.6/31
  no shutdown

interface Ethernet1/3
  description vos-leaf-sw03 [Eth1/2]
  mtu 9216
  no ip redirects
  ip address 10.0.0.10/31
  no shutdown
```
##### Конфигурация vos-leaf-sw01
```
interface Ethernet1/1
  description vos-spine-sw01 [Eth1/1]
  mtu 9216
  no ip redirects
  ip address 10.0.0.1/31
  no shutdown

interface Ethernet1/2
  description vos-spine-sw02 [Eth1/1]
  mtu 9216
  no ip redirects
  ip address 10.0.0.3/31
  no shutdown

interface Ethernet1/3
  description to Client_red
```
##### Конфигурация vos-leaf-sw02
```
interface Ethernet1/1
  description vos-spine-sw01 [Eth1/2]
  mtu 9216
  no ip redirects
  ip address 10.0.0.5/31
  no shutdown

interface Ethernet1/2
  description vos-spine-sw02 [Eth1/2]
  mtu 9216
  no ip redirects
  ip address 10.0.0.7/31
  no shutdown

interface Ethernet1/3
  description to Client_yellow
```
##### Конфигурация vos-leaf-sw03
```
interface Ethernet1/1
  description vos-spine-sw01 [Eth1/3]
  mtu 9216
  no ip redirects
  ip address 10.0.0.9/31
  no shutdown

interface Ethernet1/2
  description vos-spine-sw02 [Eth1/3]
  mtu 9216
  no ip redirects
  ip address 10.0.0.11/31
  no shutdown

interface Ethernet1/3
  description to Client_black

interface Ethernet1/4
  description to Client_white
```
### Проверка
Проверка осуществлялась при помощи ping между парой SPINE-LEAF.
Первый icmp request терялся, так как не было записи в ARP-таблице для данных адресов.
##### Проверка с vos-spine-sw01
```
vos-spine-sw01# ping 10.0.0.1
PING 10.0.0.1 (10.0.0.1): 56 data bytes
36 bytes from 10.0.0.0: Destination Host Unreachable
Request 0 timed out
64 bytes from 10.0.0.1: icmp_seq=1 ttl=254 time=56.79 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=254 time=21.341 ms
64 bytes from 10.0.0.1: icmp_seq=3 ttl=254 time=20.04 ms
64 bytes from 10.0.0.1: icmp_seq=4 ttl=254 time=11.129 ms

--- 10.0.0.1 ping statistics ---
5 packets transmitted, 4 packets received, 20.00% packet loss
round-trip min/avg/max = 11.129/27.325/56.79 ms
vos-spine-sw01# ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5): 56 data bytes
36 bytes from 10.0.0.4: Destination Host Unreachable
Request 0 timed out
64 bytes from 10.0.0.5: icmp_seq=1 ttl=254 time=26.897 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=254 time=10.663 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=254 time=9.076 ms
64 bytes from 10.0.0.5: icmp_seq=4 ttl=254 time=9.439 ms

--- 10.0.0.5 ping statistics ---
5 packets transmitted, 4 packets received, 20.00% packet loss
round-trip min/avg/max = 9.076/14.018/26.897 ms
vos-spine-sw01# ping 10.0.0.9
PING 10.0.0.9 (10.0.0.9): 56 data bytes
36 bytes from 10.0.0.8: Destination Host Unreachable
Request 0 timed out
64 bytes from 10.0.0.9: icmp_seq=1 ttl=254 time=34.61 ms
64 bytes from 10.0.0.9: icmp_seq=2 ttl=254 time=27.197 ms
64 bytes from 10.0.0.9: icmp_seq=3 ttl=254 time=18.256 ms
64 bytes from 10.0.0.9: icmp_seq=4 ttl=254 time=27.783 ms

--- 10.0.0.9 ping statistics ---
5 packets transmitted, 4 packets received, 20.00% packet loss
round-trip min/avg/max = 18.256/26.961/34.61 ms
```
##### Проверка с vos-spine-sw02
```
vos-spine-sw02# ping 10.0.0.3
PING 10.0.0.3 (10.0.0.3): 56 data bytes
36 bytes from 10.0.0.2: Destination Host Unreachable
Request 0 timed out
64 bytes from 10.0.0.3: icmp_seq=1 ttl=254 time=50.308 ms
64 bytes from 10.0.0.3: icmp_seq=2 ttl=254 time=13.075 ms
64 bytes from 10.0.0.3: icmp_seq=3 ttl=254 time=13.2 ms
64 bytes from 10.0.0.3: icmp_seq=4 ttl=254 time=17.006 ms

--- 10.0.0.3 ping statistics ---
5 packets transmitted, 4 packets received, 20.00% packet loss
round-trip min/avg/max = 13.075/23.397/50.308 ms
vos-spine-sw02# ping 10.0.0.7
PING 10.0.0.7 (10.0.0.7): 56 data bytes
36 bytes from 10.0.0.6: Destination Host Unreachable
Request 0 timed out
64 bytes from 10.0.0.7: icmp_seq=1 ttl=254 time=65.698 ms
64 bytes from 10.0.0.7: icmp_seq=2 ttl=254 time=19.042 ms
64 bytes from 10.0.0.7: icmp_seq=3 ttl=254 time=7.265 ms
64 bytes from 10.0.0.7: icmp_seq=4 ttl=254 time=14.668 ms

--- 10.0.0.7 ping statistics ---
5 packets transmitted, 4 packets received, 20.00% packet loss
round-trip min/avg/max = 7.265/26.668/65.698 ms
vos-spine-sw02# ping 10.0.0.11
PING 10.0.0.11 (10.0.0.11): 56 data bytes
36 bytes from 10.0.0.10: Destination Host Unreachable
Request 0 timed out
64 bytes from 10.0.0.11: icmp_seq=1 ttl=254 time=104.312 ms
64 bytes from 10.0.0.11: icmp_seq=2 ttl=254 time=28.671 ms
64 bytes from 10.0.0.11: icmp_seq=3 ttl=254 time=12.024 ms
64 bytes from 10.0.0.11: icmp_seq=4 ttl=254 time=20.929 ms

--- 10.0.0.11 ping statistics ---
5 packets transmitted, 4 packets received, 20.00% packet loss
round-trip min/avg/max = 12.024/41.484/104.312 ms
```
