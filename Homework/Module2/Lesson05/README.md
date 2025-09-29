### Домашнее задание в модуле №2 урок №5  Построение Underlay сети (BGP)

##### Цель задания
- Настроить BGP в Underlay сети для IP связанности между всеми сетевыми устройствами. iBGP или eBGP - решать вам!
- Зафиксировать в документации план работы, адресное пространство, схему сети, конфигурацию устройств
- Убедиться в наличии IP связанности между устройствами в BGP домене


---

### Результаты ДЗ

### **1. Топология сети IPv4 лабораторной работы в PnetLAB**:
 
 Используем уже привычную по предыдущим урокам и домашним заданиям схему сети, но на сей раз только её IPv4 вариант 

 [<img src="WEST_DC_topology_for_BGP.JPG">](https://github.com/R0gerWilco/OTUS_DC/blob/main/Homework/Module2/Lesson05/WEST_DC_topology_for_BGP.JPG)


---

### **2. Входные данные**:


- Поскольку в условиях указано "iBGP или eBGP - решать вам!", для целей этого ДЗ выбран протокол iBGP
- Общая AS в BGP домене 64777
- Loopback0 всех коммутаторов  = Router-ID в BGP домене
- Используется аутентификация BGP-соседей  с применением хеширования MD5 на PtP линках
- IPv4-адресация сохранена с предыдущей топологии,  IP-адреса коммутаторов и PtP линков указаны в [README файле первого домашнего задания](https://github.com/R0gerWilco/OTUS_DC/blob/main/Homework/Module1/Lesson03/README.md), а также отображены на схеме сети  IPv4
- Spine-коммутаторы указаны как route-reflector, Leaf-коммутаторы   - как RR Client
- Тюнинг таймеров в сторону уменьшения keelalive/hold значений до 3/9 секунд соответственно для ускорения процесса обнаружения проблем со связностью между соседями
- BFD оченьхотел, но он почему-то капризничает в лабе, не видит соседей на PtP линках и таким образом укладывает BGP соседство, если в BGP секции его активировать


---
### **3. Типовая конфигурация BGP Leaf-коммутатора на примере устройства WEST_LEAF101**
```
feature bgp

route-map REDISTRIBUTE_CONNECTED permit 10
  match interface loopback0

router bgp 64777
  router-id 10.0.0.101
  log-neighbor-changes
  address-family ipv4 unicast
    redistribute direct route-map REDISTRIBUTE_CONNECTED
    maximum-paths ibgp 8

  template peer SPINE
    remote-as 64777
    password 3 9e502c7af527f9b0
    timers 3 9
    address-family ipv4 unicast
      soft-reconfiguration inbound

  neighbor 10.201.101.1
    inherit peer SPINE
    remote-as 64777
    description SPINE_201

  neighbor 10.202.101.1
    inherit peer SPINE
    remote-as 64777
    description SPINE_202
```

### **4. Типовая конфигурация BGP Spine-коммутатора на примере устройства WEST_SPINE201**
```bash
feature bgp

router bgp 64777
  router-id 10.0.0.201
  address-family ipv4 unicast
    network 10.0.0.201/32

  neighbor 10.201.0.0/16
    remote-as 64777
    password 3 9e502c7af527f9b0
    timers 3 9
    maximum-peers 24

    address-family ipv4 unicast
      route-reflector-client
      next-hop-self all
      soft-reconfiguration inbound
```

---

### **5. Проверка таблицы BGP соседства на SPINE коммутаторах**
```bash

WEST_SPINE201# show ip bgp  sum
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.0.0.201, local AS number 64777
BGP table version is 24, IPv4 Unicast config peers 4, capable peers 3
4 network entries and 4 paths using 960 bytes of memory
BGP attribute entries [2/328], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.201.101.2    4 64777   32316   32308       24    0    0    1d02h 1                  <----------------------- LEAF 101
10.201.102.2    4 64777   32394   32382       24    0    0    1d02h 1                  <----------------------- LEAF 102
10.201.103.2    4 64777   32397   32388       24    0    0    1d02h 1                  <----------------------- LEAF 103

WEST_SPINE202# show ip bgp sum
BGP summary information for VRF default, address family IPv4 Unicast
BGP router identifier 10.0.0.202, local AS number 64777
BGP table version is 9, IPv4 Unicast config peers 4, capable peers 3
4 network entries and 4 paths using 960 bytes of memory
BGP attribute entries [2/328], BGP AS path entries [0/0]
BGP community entries [0/0], BGP clusterlist entries [0/0]

Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
10.202.101.2    4 64777   31767   31761        9    0    0    1d02h 1                   <----------------------- LEAF 101
10.202.102.2    4 64777   31777   31770        9    0    0    1d02h 1                   <----------------------- LEAF 102
10.202.103.2    4 64777   31756   31748        9    0    0    1d02h 1                   <----------------------- LEAF 103

```
---

### **6. Проверка таблицы маршрутизации на SPINE коммутаторах**

```bash

WEST_SPINE201# show ip route bgp-64777
10.0.0.101/32, ubest/mbest: 1/0                                           <----------------------- LEAF 101
    *via 10.201.101.2, [200/0], 1d02h, bgp-64777, internal, tag 64777
10.0.0.102/32, ubest/mbest: 1/0                                           <----------------------- LEAF 102
    *via 10.201.102.2, [200/0], 1d02h, bgp-64777, internal, tag 64777
10.0.0.103/32, ubest/mbest: 1/0                                           <----------------------- LEAF 103
    *via 10.201.103.2, [200/0], 1d02h, bgp-64777, internal, tag 64777


WEST_SPINE202# show ip route bgp-64777
10.0.0.101/32, ubest/mbest: 1/0                                           <----------------------- LEAF 101
    *via 10.202.101.2, [200/0], 1d02h, bgp-64777, internal, tag 64777
10.0.0.102/32, ubest/mbest: 1/0                                           <----------------------- LEAF 102
    *via 10.202.102.2, [200/0], 1d02h, bgp-64777, internal, tag 64777
10.0.0.103/32, ubest/mbest: 1/0                                           <----------------------- LEAF 103
    *via 10.202.103.2, [200/0], 1d02h, bgp-64777, internal, tag 64777
```


---

### **7. Проверка BGP маршрутов на примере LEAF коммутатора WEST_LEAF101**
```bash
WEST_LEAF101# show ip route bgp-64777 
IP Route Table for VRF "default"
'*' denotes best ucast next-hop
'**' denotes best mcast next-hop
'[x/y]' denotes [preference/metric]
'%<string>' in via output denotes VRF <string>

10.0.0.102/32, ubest/mbest: 2/0                                         <-----------------------Loopback LEAF 102 via SPINE 201 & SPINE 202
    *via 10.201.101.1, [200/0], 1d02h, bgp-64777, internal, tag 64777
    *via 10.202.101.1, [200/0], 1d02h, bgp-64777, internal, tag 64777
10.0.0.103/32, ubest/mbest: 2/0                                         <-----------------------Loopback LEAF 103 via SPINE 201 & SPINE 202
    *via 10.201.101.1, [200/0], 1d02h, bgp-64777, internal, tag 64777
    *via 10.202.101.1, [200/0], 1d02h, bgp-64777, internal, tag 64777
10.0.0.201/32, ubest/mbest: 1/0                                         <-----------------------Loopback SPINE 201 via SPINE 201
    *via 10.201.101.1, [200/0], 1d02h, bgp-64777, internal, tag 64777
10.0.0.202/32, ubest/mbest: 1/0                                         <-----------------------Loopback SPINE 202 via SPINE 202
    *via 10.202.101.1, [200/0], 1d02h, bgp-64777, internal, tag 64777
```

**Детализация маршуртной информации для сетей, доступных через multipath, на примере маршрута к Loopback LEAF102 10.0.0.102/32:**
```bash
WEST_LEAF101# show ip bgp 10.0.0.102/32
BGP routing table information for VRF default, address family IPv4 Unicast
BGP routing table entry for 10.0.0.102/32, version 14
Paths: (2 available, best #2)
Flags: (0x8008001a) (high32 00000000) on xmit-list, is in urib, is best urib rou
te, is in HW
Multipath: iBGP

  Path type: internal, path is valid, not best reason: Neighbor Address, multipa
th, no labeled nexthop, in rib
  AS-Path: NONE, path sourced internal to AS
    10.202.101.1 (metric 0) from 10.202.101.1 (10.0.0.202)
      Origin incomplete, MED 0, localpref 100, weight 0
      Originator: 10.0.0.102 Cluster list: 10.0.0.202                   <-----------------------  Originator-ID: LEAF 102 Cluster-ID: SPINE 202                                          

  Advertised path-id 1
  Path type: internal, path is valid, is best path, no labeled nexthop, in rib
  AS-Path: NONE, path sourced internal to AS
    10.201.101.1 (metric 0) from 10.201.101.1 (10.0.0.201)
      Origin incomplete, MED 0, localpref 100, weight 0
      Originator: 10.0.0.102 Cluster list: 10.0.0.201                    <-----------------------  Originator-ID: LEAF 102 Cluster-ID: SPINE 201  

```

### **8. Проверка доступности  IPv4 Loopback-интерфейсов коммутаторов фабрики с WEST_LEAF101**
```bash
WEST_LEAF101# ping 10.0.0.102 source-interface Loopback0 coun 3      <-----------------------Loopback LEAF 102
PING 10.0.0.102 (10.0.0.102): 56 data bytes
64 bytes from 10.0.0.102: icmp_seq=0 ttl=253 time=52.252 ms
64 bytes from 10.0.0.102: icmp_seq=1 ttl=253 time=31.474 ms
64 bytes from 10.0.0.102: icmp_seq=2 ttl=253 time=66.654 ms
--- 10.0.0.102 ping statistics ---
3 packets transmitted, 3 packets received, 0.00% packet loss
round-trip min/avg/max = 31.474/50.126/66.654 ms

WEST_LEAF101# ping 10.0.0.103 source-interface Loopback0 coun 3       <-----------------------Loopback LEAF 103
PING 10.0.0.103 (10.0.0.103): 56 data bytes
64 bytes from 10.0.0.103: icmp_seq=0 ttl=253 time=84.047 ms
64 bytes from 10.0.0.103: icmp_seq=1 ttl=253 time=63.98 ms
64 bytes from 10.0.0.103: icmp_seq=2 ttl=253 time=122.398 ms
--- 10.0.0.103 ping statistics ---
3 packets transmitted, 3 packets received, 0.00% packet loss
round-trip min/avg/max = 63.98/90.141/122.398 ms


WEST_LEAF101#  ping 10.0.0.201 source-interface Loopback0 coun 3       <-----------------------Loopback SPINE 201
PING 10.0.0.201 (10.0.0.201): 56 data bytes
64 bytes from 10.0.0.201: icmp_seq=0 ttl=254 time=26.45 ms
64 bytes from 10.0.0.201: icmp_seq=1 ttl=254 time=26.339 ms
64 bytes from 10.0.0.201: icmp_seq=2 ttl=254 time=32.959 ms
--- 10.0.0.201 ping statistics ---
3 packets transmitted, 3 packets received, 0.00% packet loss
round-trip min/avg/max = 26.339/28.582/32.959 ms

WEST_LEAF101#  ping 10.0.0.202 source-interface Loopback0 coun 3       <-----------------------Loopback SPINE 202
PING 10.0.0.202 (10.0.0.202): 56 data bytes
64 bytes from 10.0.0.202: icmp_seq=0 ttl=254 time=56.165 ms
64 bytes from 10.0.0.202: icmp_seq=1 ttl=254 time=60.278 ms
64 bytes from 10.0.0.202: icmp_seq=2 ttl=254 time=53.179 ms
--- 10.0.0.202 ping statistics ---
3 packets transmitted, 3 packets received, 0.00% packet loss
round-trip min/avg/max = 53.179/56.54/60.278 ms
```






