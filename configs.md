# 完整设备配置命令

---

## ISP-1 运营商路由器

```
system-view
sysname ISP-1
interface GigabitEthernet0/0/0
 ip address 172.16.1.1 255.255.255.252
 undo shutdown
 quit
interface LoopBack0
 ip address 8.8.8.8 255.255.255.255
 quit
bgp 65001
 peer 172.16.1.2 as-number 65000
 network 8.8.8.8 255.255.255.255
 quit
ip route-static 192.168.0.0 255.255.0.0 172.16.1.2
return
save
y
```

---

## ISP-2 运营商路由器

```
system-view
sysname ISP-2
interface GigabitEthernet0/0/0
 ip address 172.16.2.1 255.255.255.252
 undo shutdown
 quit
interface LoopBack0
 ip address 114.114.114.114 255.255.255.255
 quit
bgp 65002
 peer 172.16.2.2 as-number 65000
 network 114.114.114.114 255.255.255.255
 quit
ip route-static 192.168.0.0 255.255.0.0 172.16.2.2
return
save
y
```

---

## R1 出口路由器

```
system-view
sysname R1
interface GigabitEthernet0/0/0
 description TO-ISP-1
 ip address 172.16.1.2 255.255.255.252
 undo shutdown
 quit
interface GigabitEthernet0/0/1
 description TO-ISP-2
 ip address 172.16.2.2 255.255.255.252
 undo shutdown
 quit
interface GigabitEthernet0/0/2
 description TO-Core-SW
 ip address 10.0.0.1 255.255.255.252
 undo shutdown
 quit
interface LoopBack0
 ip address 1.1.1.1 255.255.255.255
 quit
ospf 1 router-id 1.1.1.1
 area 0.0.0.0
  network 10.0.0.0 0.0.0.3
  network 1.1.1.1 0.0.0.0
 quit
 quit
bgp 65000
 router-id 1.1.1.1
 peer 172.16.1.1 as-number 65001
 peer 172.16.2.1 as-number 65002
 network 192.168.10.0 255.255.255.0
 network 192.168.20.0 255.255.255.0
 network 192.168.30.0 255.255.255.0
 network 192.168.100.0 255.255.255.0
 quit
ip route-static 0.0.0.0 0.0.0.0 172.16.1.1 preference 60
ip route-static 0.0.0.0 0.0.0.0 172.16.2.1 preference 80
acl number 2000
 rule 5 permit source 192.168.10.0 0.0.0.255
 rule 10 permit source 192.168.30.0 0.0.0.255
 rule 15 permit source 192.168.100.0 0.0.0.255
 quit
interface GigabitEthernet0/0/0
 nat outbound 2000
 quit
interface GigabitEthernet0/0/1
 nat outbound 2000
 quit
ospf 1
 default-route-advertise always
 quit
return
save
y
```

---

## Core-SW 核心交换机 (S5700)

```
system-view
sysname Core-SW
vlan batch 10 20 30 99 100
interface Vlanif10
 ip address 192.168.10.254 255.255.255.0
 quit
interface Vlanif20
 ip address 192.168.20.254 255.255.255.0
 quit
interface Vlanif30
 ip address 192.168.30.254 255.255.255.0
 quit
interface Vlanif100
 ip address 192.168.100.254 255.255.255.0
 quit
interface GigabitEthernet0/0/1
 description TO-R1
 port link-type access
 port default vlan 99
 quit
interface Vlanif99
 ip address 10.0.0.2 255.255.255.252
 quit
interface GigabitEthernet0/0/2
 description TO-Agg-SW-1
 port link-type trunk
 port trunk allow-pass vlan 10 20
 quit
interface GigabitEthernet0/0/3
 description TO-Agg-SW-2
 port link-type trunk
 port trunk allow-pass vlan 20 30 100
 quit
ospf 1 router-id 10.0.0.2
 area 0.0.0.0
  network 10.0.0.0 0.0.0.3
  network 192.168.10.0 0.0.0.255
  network 192.168.20.0 0.0.0.255
  network 192.168.30.0 0.0.0.255
  network 192.168.100.0 0.0.0.255
 quit
ip route-static 0.0.0.0 0.0.0.0 10.0.0.1
aaa
 local-user admin password cipher admin@123
 local-user admin privilege level 15
 local-user admin service-type telnet ssh
 quit
user-interface vty 0 4
 authentication-mode aaa
 protocol inbound all
 quit
return
save
y
```

---

## Agg-SW-1 汇聚交换机 (S3700)

```
system-view
sysname Agg-SW-1
vlan batch 10 20 30
interface GigabitEthernet0/0/1
 description TO-Core-SW
 port link-type trunk
 port trunk allow-pass vlan 10 20 30
 quit
interface Ethernet0/0/1
 description TO-Access-1
 port link-type trunk
 port trunk allow-pass vlan 10 20 30
 quit
interface Ethernet0/0/2
 description TO-Access-2
 port link-type trunk
 port trunk allow-pass vlan 10 20 30
 quit
interface Vlanif30
 ip address 192.168.30.11 255.255.255.0
 quit
ip route-static 0.0.0.0 0.0.0.0 192.168.30.254
return
save
y
```

---

## Agg-SW-2 汇聚交换机 (S3700)

```
system-view
sysname Agg-SW-2
vlan batch 20 30 100
interface GigabitEthernet0/0/1
 description TO-Core-SW
 port link-type trunk
 port trunk allow-pass vlan 20 30 100
 quit
interface Ethernet0/0/1
 description TO-Access-3
 port link-type trunk
 port trunk allow-pass vlan 100
 quit
interface Ethernet0/0/2
 description TO-Access-4
 port link-type trunk
 port trunk allow-pass vlan 20 30
 quit
interface Vlanif30
 ip address 192.168.30.12 255.255.255.0
 quit
ip route-static 0.0.0.0 0.0.0.0 192.168.30.254
return
save
y
```

---

## Access-1 接入交换机 (S3700)

```
system-view
sysname Access-1
vlan batch 10 30
interface Ethernet0/0/1
 description TO-Agg-SW-1
 port link-type trunk
 port trunk allow-pass vlan 10 30
 quit
interface Ethernet0/0/2
 description TO-PC1
 port link-type access
 port default vlan 10
 quit
interface Vlanif30
 ip address 192.168.30.21 255.255.255.0
 quit
ip route-static 0.0.0.0 0.0.0.0 192.168.30.254
return
save
y
```

---

## Access-2 接入交换机 (S3700)

```
system-view
sysname Access-2
vlan batch 10 30
interface Ethernet0/0/1
 description TO-Agg-SW-1
 port link-type trunk
 port trunk allow-pass vlan 10 30
 quit
interface Ethernet0/0/2
 description TO-PC2
 port link-type access
 port default vlan 10
 quit
interface Vlanif30
 ip address 192.168.30.22 255.255.255.0
 quit
ip route-static 0.0.0.0 0.0.0.0 192.168.30.254
return
save
y
```

---

## Access-3 接入交换机 (S3700)

```
system-view
sysname Access-3
vlan batch 30 100
interface Ethernet0/0/1
 description TO-Agg-SW-2
 port link-type trunk
 port trunk allow-pass vlan 100 30
 quit
interface Ethernet0/0/2
 description TO-Server-PC
 port link-type access
 port default vlan 100
 quit
interface Vlanif30
 ip address 192.168.30.23 255.255.255.0
 quit
ip route-static 0.0.0.0 0.0.0.0 192.168.30.254
return
save
y
```

---

## Access-4 接入交换机 (S3700)

```
system-view
sysname Access-4
vlan batch 20 30
interface Ethernet0/0/1
 description TO-Agg-SW-2
 port link-type trunk
 port trunk allow-pass vlan 20 30
 quit
interface Ethernet0/0/2
 port link-type access
 port default vlan 20
 quit
interface Vlanif30
 ip address 192.168.30.24 255.255.255.0
 quit
ip route-static 0.0.0.0 0.0.0.0 192.168.30.254
return
save
y
```
