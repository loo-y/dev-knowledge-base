---
description: 防止ssh远程暴力攻击
---

# Preventing SSH remote brute-force attacks

#### 方法一：修改远程登录端口

修改/etc/ssh/sshd\_config文件中的Port 将前方的#注释删除，并将22修改为你想要使用远程登录的端口，例如54321。

```bash
vi /etc/ssh/sshd_config
```

删除 Port 22 前的注释，并且修改为 54321:

```bash
#Port 22
Port 54321
```

保存退出, 按ESC, 输入:

```bash
:wq
```

**最最重要的一步，千万不要忘记修改防火墙端口，否则重启ssh服务之后将无法连接！！**

```bash
# tcp 端口
firewall-cmd --permanent --add-port=54321/tcp
# udp 端口
firewall-cmd --permanent --add-port=54321/udp
# 重载防火墙
firewall-cmd --reload
```

重启 ssh 服务

```bash
systemctl restart sshd
```

#### 方法二：限制登录IP

TODO

#### 方法三：使用非root用户登录

TODO

