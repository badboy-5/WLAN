AP选择AP4030，AC选择AC6005，AP无法通过sysname更改设备名

```bash
# SW1创建VLAN
[SW1]vlan 10         
[SW1-vlan10]vlan 20
[SW1-vlan20]vlan 30

# SW1配置接口
[SW1]int g0/0/1
[SW1-GigabitEthernet0/0/1]port link-type trunk  //配置接口类型
[SW1-GigabitEthernet0/0/1]port trunk pvid vlan 10  //
[SW1-GigabitEthernet0/0/1]port trunk allow-pass vlan 10 11 12  //配置trunk接口允许通过的VLAN

[SW1]int g0/0/2
[SW1-GigabitEthernet0/0/2]port link-type trunk .
[SW1-GigabitEthernet0/0/2]port trunk pvid vlan 10
[SW1-GigabitEthernet0/0/2]port trunk llow-pass vlan 10 20 30

[SW1-GigabitEthernet0/0/2]int g0/0/3
[SW1-GigabitEthernet0/0/3]port link-type trunk 
[SW1-GigabitEthernet0/0/3]port trunk allow-pass vlan 10 20 30

# SW2建VLAN
[SW2]vlan 10
[SW2-vlan10]vlan 20
[SW2-vlan20]vlan 30
[SW2-vlan30]q

# SW2配置接口
[SW2]int g0/0/1
[SW2-GigabitEthernet0/0/1]port link-type trunk 
[SW2-GigabitEthernet0/0/1]port trunk allow-pass vlan 10 20 30
[SW2-GigabitEthernet0/0/1]int g0/0/2
[SW2-GigabitEthernet0/0/2]port link-type trunk 
[SW2-GigabitEthernet0/0/2]port trunk allow-pass vlan 10 20 30
[SW2-GigabitEthernet0/0/2]

# AC配置接口
[AC1]vlan 10
[AC1-vlan10]vlan 20
[AC1-vlan20]vlan 30
[AC1-vlan30]q
[AC1]int g0/0/1
[AC1-GigabitEthernet0/0/1]port link-type trunk 
[AC1-GigabitEthernet0/0/1]port trunk allow-pass vlan 10 20 30
[AC1-GigabitEthernet0/0/1]q
[AC1]

# 配置管理VLAN 
[AC1]int Vlanif 10
[AC1-Vlanif10]ip ad 10.1.1.1 24
[AC1-Vlanif10]q
[AC1]

# 创建AP组
[AC1]wlan 
[AC1-wlan-view]ap-group name Bad
[AC1-wlan-ap-group-Bad]

# 开启DHCP功能，基于接口的DHCP
[AC1]dhcp enable 
[AC1]int Vlanif 10
[AC1-Vlanif10]dhcp select interface 
[AC1-Vlanif10]int vlan20
[AC1-Vlanif20]ip ad 20.1.1.1 24
[AC1-Vlanif20]dhcp select interface
[AC1-Vlanif20]int vlan30
[AC1-Vlanif30]ip ad 30.1.1.1 24
[AC1-Vlanif30]dhcp select interface

# 创建管理域并配置AC国家码
[AC1]wlan
[AC1-wlan-view]regulatory-domain-profile name YGL  //创建管理域模板
[AC1-wlan-regulate-domain-YGL]country-code CN  //配置AC国家码
[AC1-wlan-regulate-domain-YGL]q
[AC1-wlan-view]ap-group name Bad  //域管理绑定至AP组
[AC1-wlan-ap-group-Bad]regulatory-domain-profile YGL

# 建立隧道
[AC1]capwap source interface Vlanif 10

# 配置AP1
[AC1]wlan
[AC1-wlan-view]ap auth-mode mac-auth  //使用MAC地址认证
[AC1-wlan-view]ap-mac 00E0-FC9D-2A60 ap-id 0  //配置认证
[AC1-wlan-ap-0]ap-group Bad  //添加至AP组
Warning: This operation may cause AP reset. If the country code changes, it will clear channel, power and antenna gain configurations of the radio, Whether to continue? [Y/N]:y  //Y确认

# 配置AP2
[AC1-wlan-view]ap-mac 00E0-FCDC-66D0 ap-id 1  
[AC1-wlan-ap-1]ap-group Bad  //绑定AP组
Warning: This operation may cause AP reset. If the country code changes, it will clear channel, power and antenna gain configurations of the radio, Whether to continue? [Y/N]:y  //Y确认
```

- 配置VAP模板

**安全模板**

```bash
[AC1-wlan-view]security-profile name 1  //配置安全模板
[AC1-wlan-sec-prof-1]q 
[AC1-wlan-view] 
```

**配置SSID模板**

```bash
# 创建SSID
[AC1-wlan-view]ssid-profile name 1
[AC1-wlan-ssid-prof-1]ssid 1
[AC1-wlan-ssid-prof-1]q
[AC1-wlan-view]ssid-profile name 2
[AC1-wlan-ssid-prof-2]ssid 2
[AC1-wlan-ssid-prof-2]q
[AC1-wlan-view]

# VAP模板调用SSID模板
[AC1-wlan-view]vap-profile name 1
[AC1-wlan-vap-prof-1]forward-mode direct-forward  //直接转发模式
[AC1-wlan-vap-prof-1]service-vlan vlan-id 20  //指定业务WLAN
[AC1-wlan-vap-prof-1]security-profile 1  //调用安全模板
[AC1-wlan-vap-prof-1]ssid-profile 1  //调用SSID模板
[AC1-wlan-vap-prof-1]q

[AC1-wlan-view]vap-profile name 2
[AC1-wlan-vap-prof-2]forward-mode direct-forward 
[AC1-wlan-vap-prof-2]service-vlan vlan-id 30
[AC1-wlan-vap-prof-2]security-profile 1
[AC1-wlan-vap-prof-2]ssid-profile 2
[AC1-wlan-vap-prof-2]

# 加入所有模板
[AC1-wlan-view]ap-group name Bad
[AC1-wlan-ap-group-Bad]vap-profile 1 wlan 1 radio all 
[AC1-wlan-ap-group-Bad]vap-profile 2 wlan 2 radio all 
[AC1-wlan-ap-group-Bad]
```