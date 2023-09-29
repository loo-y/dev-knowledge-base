# Ubuntu安装Netflix-Proxy

需要先安装docker，不要直接用Netflix-proxy github里安装docker的方法，那个有问题！！

安装docker的方法参考[docker官方](https://docs.docker.com/engine/install/ubuntu/)

### 安装官方docker

1. 确认移除残留的docker工具

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

1. 设置源

```bash
sudo apt-get update
```

```bash

sudo apt-get install \\
    ca-certificates \\
    curl \\
    gnupg \\
    lsb-release
```

```bash
curl -fsSL <https://download.docker.com/linux/ubuntu/gpg> | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```bash
echo \\
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] <https://download.docker.com/linux/ubuntu> \\
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

1. 再次更新以及安装docker文件

```bash
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

1. 验证helloworld

```bash
sudo docker run hello-world
```

1. 安装dig以及nslookup

```bash
apt-get install dnsutils
```

### 安装Netflix-proxy

**注意，如果安装失败，看一下netflix-proxy.log，可能是源不对，需要把错误的源删除！！**

参考[Netflix-proxy Github](https://github.com/ab77/netflix-proxy)

```bash
mkdir -p ~/netflix-proxy\\
  && cd ~/netflix-proxy\\
  && curl -fsSL <https://github.com/ab77/netflix-proxy/archive/latest.tar.gz> | gunzip - | tar x --strip-components=1\\
  && ./build.sh
```

### 打开防火墙

Ubuntu默认iptables会不允许访问，需要添加防火墙设置

1. 安装iptables-persistent

```bash
sudo apt-get install iptables-persistent
```

1. 设置允许访问端口

```bash
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
iptables -I INPUT -p tcp --dport 443 -j ACCEPT
iptables -I INPUT -p tcp --dport 53 -j ACCEPT
iptables -I INPUT -p udp --dport 53 -j ACCEPT
```

1. 持久化规则

```bash
sudo netfilter-persistent save

sudo netfilter-persistent reload
```

### 最后

访问站点，添加IP
