### Домашнее задание в модуле №2 урок №3  Построение Underlay сети (IS-IS)

##### Цель задания
- Настроить ISIS в Underlay сети для IP связанности между всеми сетевыми устройствами.
- Зафиксировать в документации план работы, адресное пространство, схему сети, конфигурацию устройств
- Убедиться в наличии IP связанности между устройствами в ISIS домене


---

### Результаты ДЗ

### **1. Топология сети IPv4 лабораторной работы в PnetLAB**:

 [<img src="WEST_DC_topology_for_ISIS.JPG">](https://github.com/R0gerWilco/OTUS_DC/blob/main/Homework/Module2/Lesson03/WEST_DC_topology_for_ISIS.JPG)


### **2. Топология сети IPv6 лабораторной работы в PnetLAB**:

 [<img src="WEST_DC_topology_for_ISIS_IPv6.JPG">](https://github.com/R0gerWilco/OTUS_DC/blob/main/Homework/Module2/Lesson03/WEST_DC_topology_for_ISIS_IPv6.JPG)



---

### **3. Входные данные**:

#### **3.1. Общая информация**

- Общая Area 49.001
- Отношения соседства ISIS на PtP линках указаны как Level2
- Используется аутентификация с применением хеширования MD5 на PtP линках
- IPv4-адресация сохранена с предыдущей топологии,  IP-адреса коммутаторов и PtP линков указаны в [README файле первого домашнего задания](https://github.com/R0gerWilco/OTUS_DC/blob/main/Homework/Module1/Lesson03/README.md), а также отображены на схеме сети  IPv4
- IPv6-адреса коммутаторов и PtP линков указаны в таблицах ниже, а также отображены на схеме сети  IPv6

#### **3.2. Таблица Loopback в IPv6**
| Устройство        | Loopback (IPv6)  | 
|-------------------|------------------|
| **WEST_SPINE201** | `10::201/128`    | 
| **WEST_SPINE202** | `10::202/128`    | 
| **WEST_LEAF101**  | `10::101/128`    | 
| **WEST_LEAF102**  | `10::102/128`    |  
| **WEST_LEAF103**  | `10::103/128`    | 


#### **3.3 Таблица P2P соединений в IPv6**
| Соединение (w/o DC Name)| Spine-адрес         | Leaf-адрес          | Подсеть              |
|-------------------------|---------------------|---------------------|----------------------|
| **Spine201 ↔ Leaf101**  | `10:201:101::1/127` | `10:201:101::2/127` | `10:201:101::0/127`  |
| **Spine201 ↔ Leaf102**  | `10:201:102::1/127` | `10:201:102::2/127` | `10:201:102::0/127`  |
| **Spine201 ↔ Leaf103**  | `10:201:103::1/127` | `10:201:103::1/127` | `10:201:103::0/127`  |
| **Spine202 ↔ Leaf101**  | `10:202:101::1/127` | `10:202:101::2/127` | `10:202:101::0/127`  |
| **Spine202 ↔ Leaf102**  | `10:202:102::1/127` | `10:202:102::2/127` | `10:202:102::0/127`  |
| **Spine202 ↔ Leaf103**  | `10:202:103::1/127` | `10:202:103::2/127` | `10:202:103::0/127`  |




---
### **4. Типовая конфигурация ISIS на примере коммутатора WEST_LEAF101**
```bash
feature isis

  interface Ethernet1/6
  description TO_SPINE201
  no switchport
  no ip redirects
  ip address 10.201.101.2/30
  ip verify unicast source reachable-via rx
  ipv6 address 10:201:101::2/127
  no ipv6 redirects
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication-type md5 level-2
  isis authentication key-chain ISISKey
  ip router isis UNDERLAY
  ipv6 router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

interface Ethernet1/7
  description TO_SPINE202
  no switchport
  no ip redirects
  ip address 10.202.101.2/30
  ip verify unicast source reachable-via rx
  ipv6 address 10:202:101::2/127
  no ipv6 redirects
  isis network point-to-point
  isis circuit-type level-2
  isis authentication-type md5
  isis authentication-type md5 level-2
  isis authentication key-chain ISISKey
  ip router isis UNDERLAY
  ipv6 router isis UNDERLAY
  no isis passive-interface level-2
  no shutdown

nterface loopback0
  description LoopBack_LEAF101
  ip address 10.0.0.101/32
  ipv6 address 10::101/128
  ip router isis UNDERLAY
  ipv6 router isis UNDERLAY

router isis UNDERLAY
  net 49.0001.0000.0000.0101.00
  is-type level-2
  address-family ipv4 unicast
  address-family ipv6 unicast
```

### **5. Типичное состояние процесса ISIS на примере коммутатора WEST_LEAF101**
```bash
WEST_LEAF101# show isis UNDERLAY 
ISIS process : UNDERLAY
 Instance number :  1
 UUID: 1090519320
 Process ID 14477
VRF: default
  System ID : 0000.0000.0101  IS-Type : L2
  SAP : 412  Queue Handle : 13
  Maximum LSP MTU: 1492
  Stateful HA enabled
  Graceful Restart enabled. State: Inactive 
  Last graceful restart status : none
  Start-Mode Complete
  BFD IPv4 is globally disabled for ISIS process: UNDERLAY
  BFD IPv6 is globally disabled for ISIS process: UNDERLAY
  Topology-mode is base
  Metric-style : advertise(wide), accept(narrow, wide)
  Area address(es) :
    49.0001
  Process is up and running
  VRF ID: 1
  Stale routes during non-graceful controlled restart
  Enable resolution of L3->L2 address for ISIS adjacency
  SRTE: Not registered
  OAM: Not registered
  SR IPv4 is not configured and disabled for ISIS process: UNDERLAY
  SR IPv6 is not configured and disabled for ISIS process: UNDERLAY
SRv6 feature not present
  Interfaces supported by IS-IS :
    loopback0
    Ethernet1/6
    Ethernet1/7
  Topology : 0
  Address family IPv4 unicast :
    Number of interface : 3
    Distance : 115
    Default-information not originated
  Address family IPv6 unicast :
    Number of interface : 3
    Distance : 115
    Default-information not originated
  Topology : 2
  Address family IPv4 unicast :
    Number of interface : 0
    Distance : 115
    Default-information not originated
  Address family IPv6 unicast :
    Number of interface : 0
    Distance : 115
    Default-information not originated
  Level1
  No auth type and keychain
  Auth check set
  Level2
  No auth type and keychain
  Auth check set
  L1 Next SPF: Inactive
  L2 Next SPF: Inactive
  Attached bits
   MT-0 L-1: Att 0 Spf-att 0 Cfg 1 Adv-att 0
   MT-0 L-2: Att 0 Spf-att 0 Cfg 1 Adv-att 0
```

---

### **6. Проверка таблицы ISIS соседства на SPINE коммутаторах**
```bash
WEST_SPINE201# show isis adjacency 
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
WEST_LEAF101    N/A             2      UP     00:00:26   Ethernet1/1       <----------------------- LEAF 101
WEST_LEAF102    N/A             2      UP     00:00:30   Ethernet1/2       <----------------------- LEAF 102
WEST_LEAF103    N/A             2      UP     00:00:26   Ethernet1/3       <----------------------- LEAF 103

WEST_SPINE202# show isis adjacency 
IS-IS process: UNDERLAY VRF: default
IS-IS adjacency database:
Legend: '!': No AF level connectivity in given topology
System ID       SNPA            Level  State  Hold Time  Interface
WEST_LEAF101    N/A             2      UP     00:00:32   Ethernet1/1        <----------------------- LEAF 101
WEST_LEAF102    N/A             2      UP     00:00:32   Ethernet1/2        <----------------------- LEAF 102
WEST_LEAF103    N/A             2      UP     00:00:28   Ethernet1/3        <----------------------- LEAF 103
```


---

### **5. Проверка таблицы маршрутизации на примере LEAF коммутатора WEST_LEAF101**
```bash
WEST_LEAF101# show ip route ospf

10.0.0.102/32, ubest/mbest: 2/0                            <-----------------------Loopback LEAF 102 via SPINE 201 & SPINE 202
    *via 10.201.101.1, Eth1/6, [110/81], 01:38:18, ospf-UNDERLAY, intra
    *via 10.202.101.1, Eth1/7, [110/81], 01:38:14, ospf-UNDERLAY, intra
10.0.0.103/32, ubest/mbest: 2/0                            <-----------------------Loopback LEAF 103 via SPINE 201 & SPINE 202
    *via 10.201.101.1, Eth1/6, [110/81], 01:38:20, ospf-UNDERLAY, intra
    *via 10.202.101.1, Eth1/7, [110/81], 01:38:14, ospf-UNDERLAY, intra
10.0.0.201/32, ubest/mbest: 1/0                            <-----------------------Loopback SPINE 201 via SPINE 201
    *via 10.201.101.1, Eth1/6, [110/41], 01:38:22, ospf-UNDERLAY, intra
10.0.0.202/32, ubest/mbest: 1/0                            <-----------------------Loopback SPINE 202 via SPINE 202 
    *via 10.202.101.1, Eth1/7, [110/41], 01:38:14, ospf-UNDERLAY, intra
10.201.102.0/30, ubest/mbest: 1/0                          <-----------------------PtP LEAF 102 - SPINE 201 via SPINE 201
    *via 10.201.101.1, Eth1/6, [110/80], 01:38:22, ospf-UNDERLAY, intra
10.201.103.0/30, ubest/mbest: 1/0                           <-----------------------PtP LEAF 103 - SPINE 201 via SPINE 201
    *via 10.201.101.1, Eth1/6, [110/80], 01:38:22, ospf-UNDERLAY, intra
10.202.102.0/30, ubest/mbest: 1/0                           <-----------------------PtP LEAF 102 - SPINE 202 via SPINE 202
    *via 10.202.101.1, Eth1/7, [110/80], 01:38:14, ospf-UNDERLAY, intra
10.202.103.0/30, ubest/mbest: 1/0                           <-----------------------PtP LEAF 103 - SPINE 202 via SPINE 202
    *via 10.202.101.1, Eth1/7, [110/80], 01:38:14, ospf-UNDERLAY, intra
```

### **6. Проверка доступности Loopback-интерфейсов коммутаторов фабрики с WEST_LEAF101**
```bash
WEST_LEAF101# ping 10.0.0.102 source-interface Loopback0                  <-----------------------Loopback LEAF 102
PING 10.0.0.102 (10.0.0.102): 56 data bytes
64 bytes from 10.0.0.102: icmp_seq=0 ttl=253 time=18.869 ms
64 bytes from 10.0.0.102: icmp_seq=1 ttl=253 time=17.004 ms
64 bytes from 10.0.0.102: icmp_seq=2 ttl=253 time=11.81 ms
64 bytes from 10.0.0.102: icmp_seq=3 ttl=253 time=8.608 ms
64 bytes from 10.0.0.102: icmp_seq=4 ttl=253 time=16.892 ms
--- 10.0.0.102 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 8.608/14.636/18.869 ms

WEST_LEAF101# ping 10.0.0.103 source-interface Loopback0                   <-----------------------Loopback LEAF 103
PING 10.0.0.103 (10.0.0.103): 56 data bytes
64 bytes from 10.0.0.103: icmp_seq=0 ttl=253 time=74.995 ms
64 bytes from 10.0.0.103: icmp_seq=1 ttl=253 time=30.342 ms
64 bytes from 10.0.0.103: icmp_seq=2 ttl=253 time=17.605 ms
64 bytes from 10.0.0.103: icmp_seq=3 ttl=253 time=15.329 ms
64 bytes from 10.0.0.103: icmp_seq=4 ttl=253 time=33.473 ms
--- 10.0.0.103 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 15.329/34.348/74.995 ms

WEST_LEAF101# ping 10.0.0.201 source-interface Loopback0                   <-----------------------Loopback SPINE 201
PING 10.0.0.201 (10.0.0.201): 56 data bytes
64 bytes from 10.0.0.201: icmp_seq=0 ttl=254 time=9.268 ms
64 bytes from 10.0.0.201: icmp_seq=1 ttl=254 time=15.892 ms
64 bytes from 10.0.0.201: icmp_seq=2 ttl=254 time=5.289 ms
64 bytes from 10.0.0.201: icmp_seq=3 ttl=254 time=12.943 ms
64 bytes from 10.0.0.201: icmp_seq=4 ttl=254 time=6.483 ms
--- 10.0.0.201 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 5.289/9.975/15.892 ms

WEST_LEAF101# ping 10.0.0.202 source-interface Loopback0                   <-----------------------Loopback SPINE 202
PING 10.0.0.202 (10.0.0.202): 56 data bytes
64 bytes from 10.0.0.202: icmp_seq=0 ttl=254 time=11.667 ms
64 bytes from 10.0.0.202: icmp_seq=1 ttl=254 time=10.754 ms
64 bytes from 10.0.0.202: icmp_seq=2 ttl=254 time=11.429 ms
64 bytes from 10.0.0.202: icmp_seq=3 ttl=254 time=9.959 ms
64 bytes from 10.0.0.202: icmp_seq=4 ttl=254 time=29.177 ms
--- 10.0.0.202 ping statistics ---
5 packets transmitted, 5 packets received, 0.00% packet loss
round-trip min/avg/max = 9.959/14.597/29.177 ms
```
