
# USG6000V Web登录管理

- Cloud配置双通道
- 防火墙
	* 初始用户名：admin
	* 初始密码：Admin@123

> 第一次登录需要更改密码，第一次输入`Admin@123`，然后输入两次新密码（可以设为huawei123）

```bash
[USG6000V1]int g0/0/0
[USG6000V1-GigabitEthernet0/0/0]ip add 192.168.43.3 24
[USG6000V1-GigabitEthernet0/0/0]service-manage all permit
```

> Web登录验证，地址栏输入：`https://192.168.43.3`或`https://192.168.43.3:8443`登录

# 实验：配置OSPF、RIP、单臂路由

- 按照图示，配置接口地址

```bash
# R1
[AR1]int g0/0/0
[AR1-GigabitEthernet0/0/0]ip ad 12.1.1.1 24
[AR1-GigabitEthernet0/0/0]int lo 1
[AR1-LoopBack1]ip ad 1.1.1.1 32
[AR1-LoopBack1]q
[AR1]

# R2
[AR2]int lo 1
[AR2-LoopBack1]ip ad 2.2.2.2 32
[AR2-LoopBack1]int g0/0/0
[AR2-GigabitEthernet0/0/0]ip ad 12.1.1.2 24
[AR2-GigabitEthernet0/0/0]int g0/0/1
[AR2-GigabitEthernet0/0/1]ip ad 23.1.1.2 24
[AR2-GigabitEthernet0/0/1]q
[AR2]

# R3
[AR3]int lo 1
[AR3-LoopBack1]ip ad 3.3.3.3 32
[AR3-LoopBack1]int g0/0/0 
[AR3-GigabitEthernet0/0/0]ip ad 23.1.1.3 24
[AR3-GigabitEthernet0/0/1]q
[AR3]

# R4
[AR4]int g0/0/0
[AR4-GigabitEthernet0/0/0]ip ad 14.1.1.4 24
[AR4-GigabitEthernet0/0/0]int lo 1
[AR4-LoopBack1]ip ad 4.4.4.4 32
[AR4-LoopBack1]q
[AR4]
```

- 配置OSPF和RIP

```bash
# R1配置OSPF
[AR1]ospf router-id 1.1.1.1
[AR1-ospf-1]import-route static 
[AR1-ospf-1]import-route rip 
[AR1-ospf-1]area 0
[AR1-ospf-1-area-0.0.0.0]net 1.1.1.1 0.0.0.0
[AR1-ospf-1-area-0.0.0.0]net 12.1.1.0 0.0.0.255 

[AR1-GigabitEthernet0/0/0]ospf network-type p2p

# R1配置RIP
[AR1]rip 1
[AR1-rip-1]version 2
[AR1-rip-1]network 4.0.0.0
[AR1-rip-1]network 14.0.0.0
[AR1-rip-1]import-route ospf 1

# R2配置OSPF
[AR2]ospf router-id 2.2.2.2
[AR2-ospf-1]area 0
[AR2-ospf-1-area-0.0.0.0]net 2.2.2.2 0.0.0.0
[AR2-ospf-1-area-0.0.0.0]net 12.1.1.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR2-ospf-1-area-0.0.0.0]q

[AR2-GigabitEthernet0/0/0]ospf network-type p2p
[AR2-GigabitEthernet0/0/1]ospf network-type p2p

# R3配置OSPF
[AR3]ospf router-id 3.3.3.3    
[AR3-ospf-1]import-route rip 
[AR3-ospf-1]area 0
[AR3-ospf-1-area-0.0.0.0]net 3.3.3.3 0.0.0.0
[AR3-ospf-1-area-0.0.0.0]net 23.1.1.0 0.0.0.255
[AR3-ospf-1-area-0.0.0.0]q

[AR3-GigabitEthernet0/0/0]ospf network-type p2p

# R3配置RIP
[AR3]rip 1
[AR3-rip-1]version 2
[AR3-rip-1]network 192.168.10.0
[AR3-rip-1]network 192.168.20.0
[AR3-rip-1]import-route ospf 1
```

- R1与R4互相通信

```bash
# R4配置RIP
[AR4]rip 1
[AR4-rip-1]version 2
[AR4-rip-1]network 4.0.0.0
[AR4-rip-1]network 14.0.0.0

# R4配置静态路由
[AR4]ip route-static 14.1.1.0 255.255.255.0 14.1.1.1
```

- 两台交换机之间配置互相通信

```bash
# SW1
[SW1]vlan 10
[SW1-vlan 10]vlan 20
[SW1-vlan20]int g0/0/2 
[SW1-GigabitEthernet0/0/2]port link-type trunk      
[SW1-GigabitEthernet0/0/2]port trunk allow-pass vlan 10 20     
[SW1-GigabitEthernet0/0/2]int g0/0/3
[SW1-GigabitEthernet0/0/3]port link-type access 
[SW1-GigabitEthernet0/0/3]port default vlan 10
[SW1-GigabitEthernet0/0/3]int g0/0/4
[SW1-GigabitEthernet0/0/4]port link-type access 
[SW1-GigabitEthernet0/0/4]port default vlan 20
[SW1-GigabitEthernet0/0/4]q
[SW1]

# SW2
[SW2]vlan 10
[SW2-vlan10]vlan 20
[SW2-vlan20]int G0/0/1
[SW2-GigabitEthernet0/0/1]port link-type trunk 
[SW2-GigabitEthernet0/0/1]port trunk allow-pass vlan 10 20
[SW2-GigabitEthernet0/0/1]int g0/0/2
[SW2-GigabitEthernet0/0/2]port link-type access 
[SW2-GigabitEthernet0/0/2]port default vlan 10
[SW2-GigabitEthernet0/0/2]int g0/0/3
[SW2-GigabitEthernet0/0/3]port link-type access 
[SW2-GigabitEthernet0/0/3]port default vlan 20
[SW2-GigabitEthernet0/0/3]q
[SW2]
```

- 配置单臂路由

```bash
# SW1
[SW1]int g0/0/1 
[SW1-GigabitEthernet0/0/1]port link-type trunk      
[SW1-GigabitEthernet0/0/1]port trunk allow-pass vlan 10 20  


# AR3
[AR3]int g0/0/1.10
[AR3-GigabitEthernet0/0/1.10]dot1q termination vid 10
[AR3-GigabitEthernet0/0/1.10]arp broadcast enable
[AR3-GigabitEthernet0/0/1.10]ip ad 192.168.10.254 24
[AR3-GigabitEthernet0/0/1.10]int g0/0/1.20
[AR3-GigabitEthernet0/0/1.20]dot1q termination vid 20
[AR3-GigabitEthernet0/0/1.20]arp broadcast enable
[AR3-GigabitEthernet0/0/1.20]ip ad 192.168.20.254 24
```

- 连通性测试