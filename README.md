# 企业网络拓扑设计与实现

> 基于华为 eNSP 的中型企业三层网络架构设计

## 项目概述

设计并实现了一个包含 10 台设备的中型企业网络：三层架构、4个VLAN、OSPF动态路由、BGP双线接入、ACL+NAT安全策略。

## 文件说明

| 文件 | 内容 |
|------|------|
| network-design.md | 完整设计方案 |
| configs.md | 所有设备配置命令 |
| 123.topo | eNSP拓扑文件 |

## 关键成果

- OSPF邻居Full状态 ✓
- BGP双ISP Established ✓
- 跨VLAN通信正常 ✓
- 四个VLAN网关全部UP ✓
