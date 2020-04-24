# 配置Stelnet登录

R1为客户机，R2为服务器

- 在R2上的配置

```bash
# 接口配置IP地址
<Huawei>sy
[Huawei]sys FWQ
[FWQ]int g0/0/0 
[FWQ-GigabitEthernet0/0/0]ip add 2.2.2.29 24
[FWQ-GigabitEthernet0/0/0]q
# 生本地rsa密钥
[FWQ]rsa local-key-pair create  //创建密钥
The key name will be: Host
% RSA keys defined for Host already exist.
Confirm to replace them? (y/n)[n]:y  //y确认
The range of public key size is (512 ~ 2048).
NOTES: If the key modulus is greater than 512,
       It will take a few minutes.
Input the bits in the modulus[default = 512]:512  //密钥长度
Generating keys...
................................................++++++++++++
..........++++++++++++
.++++++++
........++++++++
# 配置AAA认证
[FWQ]aaa
[FWQ-aaa]local-user bad password cipher huawei@123  //创建用户及密码
Info: Add a new user.
[FWQ-aaa]local-user bad service-type ssh  //配置用户允许登录方式
[FWQ-aaa]local-user bad privilege level 5  //设置账户等级
[FWQ-aaa]q
[FWQ]user-interface vty 0 4  //配置允许用户登录
[FWQ-ui-vty0-4]authentication-mode aaa  //用户登录的方式
[FWQ-ui-vty0-4]protocol inbound ssh //允许通过ssh登录
[FWQ-ui-vty0-4]q  
# 在系统视图下创建一个用户，指定ssh登录方式为密码登录
[FWQ]ssh user bad authentication-type password  //配置密码登录
 Authentication type setted, and will be in effect next time
[FWQ]stelnet server enable  //开启Stelnet服务
```

- 在R1上的配置

```bash
#配置IP
<Huawei>sy
Enter system view, return user view with Ctrl+Z.
[Huawei]sys KH
[KH]int g0/0/0
[KH-GigabitEthernet0/0/0]ip add 2.2.2.2 24
# 开启首次认证
[KH]ssh client first-time enable 
# Stelnet登录
[KH]stelnet 2.2.2.29
Please input the username:wangjie  //用户名
Trying 2.2.2.29 ...
Press CTRL+K to abort
Connected to 2.2.2.29 ...
The server is not authenticated. Continue to access it? (y/n)[n]:y  //y确认
Apr  2 2020 20:25:28-08:00 KH %%01SSH/4/CONTINUE_KEYEXCHANGE(l)[0]:The server had not been authenticated in the process of exchanging keys. When deciding whether to continue, the user chose Y. 
[KH]
Save the server's public key? (y/n)[n]:y  //y确认
The server's public key will be saved with the name 2.2.2.29. Please wait...

Apr  2 2020 20:25:30-08:00 KH %%01SSH/4/SAVE_PUBLICKEY(l)[1]:When deciding whether to save the server's public key 2.2.2.29, the user chose Y. 
[KH]
Enter password:   //密码
<FWQ>
```

