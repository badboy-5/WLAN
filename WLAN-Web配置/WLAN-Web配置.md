---
姓名：坏坏
实验时间：2020年4月24日
整理时间：2020年4月24日
---

参看博客：[【CSDN】](https://blog.csdn.net/qq_45668124/article/details/105722265)

## 交换机及AC配置

```bash
# SW1配置
[SW1]vlan batch 10 20 30
[SW1]int g0/0/1
[SW1-GigabitEthernet0/0/1]port link-type trunk 
[SW1-GigabitEthernet0/0/1]port trunk pvid vlan 10
[SW1-GigabitEthernet0/0/1]port trunk allow-pass vlan 10 20
[SW1-GigabitEthernet0/0/1]int g0/0/2
[SW1-GigabitEthernet0/0/2]port link-type trunk 
[SW1-GigabitEthernet0/0/2]port trunk allow-pass vlan 10 20

# SW2配置
[SW2]vlan 10 20 30
[SW2]int g0/0/2	
[SW2-GigabitEthernet0/0/2]port link-type trunk 
[SW2-GigabitEthernet0/0/2]port trunk allow-pass vlan 10 20
[SW2-GigabitEthernet0/0/2]int g0/0/1	
[SW2-GigabitEthernet0/0/1]port link-type trunk 	
[SW2-GigabitEthernet0/0/1]port trunk allow-pass vlan 10 20

# AC配置
[AC1]vlan batch 10 20 30
[AC1]vlan 5

[AC1]int vlan	
[AC1]int Vlanif 5
[AC1-Vlanif5]ip ad 192.168.43.2 24

[AC1]int g0/0/1
[AC1-GigabitEthernet0/0/1]port link-type trunk 
[AC1-GigabitEthernet0/0/1]port trunk allow-pass vlan 10 20
[AC1-GigabitEthernet0/0/1]int g0/0/2
[AC1-GigabitEthernet0/0/2]port link-type access 	
[AC1-GigabitEthernet0/0/2]port default vlan 5
```