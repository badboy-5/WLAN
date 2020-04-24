【参考博客】
[【CSDN】](https://blog.csdn.net/qq_45668124/article/details/105258947)

# Telnet 登录和web登录

```bash
<AC6605>system-view 
[AC6605]sys AC1 
[AC1]vlan 10  //创建VLAN
[AC1]int Vlanif 10
[AC1-Vlanif10]ip address 192.168.43.120 255.255.255.0  //配置IP地址
[AC1-Vlanif10]q
[AC1]int g0/0/1
[AC1-GigabitEthernet0/0/1]port link-type access  //配合接口类型
[AC1-GigabitEthernet0/0/1]port default vlan 10  //将接口划分进VLAN10
[AC1-GigabitEthernet0/0/1]dis ip int bri //查看配置
Vlanif10                          192.168.43.120/24    up         up     

[AC1]ping 192.168.43.1  //连通性测试

[AC1]telnet server enable  //开启Telnet服务
[AC1]aaa  //配置aaa
[AC1-aaa]local-user bad password cipher huawei@123  //创建用户并设置密码
[AC1-aaa]local-user bad service-type telnet  //设置账户类型
[AC1-aaa]local-user bad privilege level 5  //设置等级
Warning: This operation may affect online users, are you sure to change the user privilege level ?[Y/N]y
[AC1-aaa]q

[AC1]user-interface vty 0 4
[AC1-ui-vty0-4]protocol inbound all  // 允许登录接入用户类型的协议
[AC1-ui-vty0-4]authentication-mode aaa  //修改aaa认证模式
[AC1-ui-vty0-4]return

<AC1>telnet 192.168.43.120  //Telnet登录
Login authentication

Username:bad
Password:
<AC1>

[AC1]http server enable  //开启web登录
```

# Web登录管理

- 览器地址栏输入：`https://192.168.43.120`

> 用户名：admin
> 默认密码：admin@huawei.com

- 修改默认的密码就OK了!
