# SNIProxy+Dnsmasq实现Netflix-Proxy

一般来说，只要通过[一键安装Netflix-Proxy](ubuntu-an-zhuang-netflixproxy.md)即可实现，但是在部分场景，比如Nat中转的机器，没有对外暴露IPv4的vps，以及openVZ的机器，都无法通过一键实现，同时也了解一下Netflix-Proxy背后实际的实现方式。

我们要做的是，在一台有原生IP，或者可以访问Netflix/DisneyPlus的VPS上，搭建Dnsmasq和SNIProxy来供其它VPS做跳板。

比如，有一台H机搭建了Hysteria，另外一台X机搭建了SNIProxy+Dnsmasq做跳板。现在实际有两种实现方式

#### A方案

1. 需要在H机上将所有Netflix的站点的DNS解析都指向X机，这里不管是使用AdGuard，或者直接修改H机上/etc/下dns的解析都可以。
2. X机上的SNIProxy会listen对外暴露的80端口和443端口，然后做反向代理，SNIProxy可以设置table来实现对哪些域名做反向代理。
3. X机上的SNIProxy在反代Netflix之后，再返回给H机，就实现了H跳X访问Netflix解锁。

#### B方案

1. H机上的DNS的nameserver统一改为X机，同样，不管是AdGuard还是直接修改/etc/下的dns解析，只留一个nameserver为X机。
2. 这样，H机上的DNS解析就全部交给X机了，X机的Dnsmasq在接收到H机的解析请求，也就是53端口的访问，会进行劫持，并且在Dnsmasq通过配置对Netflix的访问统统都指向X机自己。
3. X机的SNIProxy在接收到Netflix访问之后进行反代。这里和A方案的第2步第3步相同。

A方案和B方案相同的地方是，X机的SNIProxy处理是一样的，区别就在谁来对Netflix做DNS解析。

**A方案：H机访问Netflix —> H机解析Netflix为X机 —> H机认为Netflix为X机并进行访问 —> X机SNIProxy反代Netflix**

**B方案：H机访问Netflix —> H机发现DNS服务器为X机 —> H机将Netflix域名发送给X机 —> X机Dnsmasq进行劫持 —> X机将Netflix解析为自己(X机)并返回给H机 —> H机认为Netflix为X机并进行访问 —> X机SNIProxy反代Netflix**

在这里可以看出，A方案实际已经不需要X机上安装Dnsmasq做域名解析，直接简单粗暴地在H机上将特定域名或者IP指向X机即可。

#### 关于Dnsmasq配置的一点疑问和解惑

在X机上配置了Dnsmasq之后，实际X机本身已经作为一台DNS服务器了，但这个配置并不会对X机自身正常访问网站进行干扰。

在B方案中，H机请求X机的53端口，也就是请求了X机的Dnsmasq服务。但X机自己的outbound出口还是走的正常的DNS解析，这里可以查看 **/etc/resolv.conf** 进行确认。如果 /etc/resolv.conf 下也将X机自身的IP作为DNS，并且同时也在Dnsmasq下配置了Netflix指向自己，那将会是一个死循环！

Dnsmasq其本身的作用并不是为了给其它机器做DNS服务，而是将有干扰的域名进行清理，所以常规的用法反而是，在 **/etc/resolv.conf** 下将自己的IP作为DNS服务器，自己的Dnsmasq来接管所有的outbound DNS解析，然后再在Dnsmasq的配置下，将域名指向想要的IP，而非指向自己的IP！

而这篇文章的搭配仅仅是将X机作为一台转发服务器来供外界使用，所以不能把自己的outbound再进行二次处理了，否则会产生循环风暴！

### Dnsmasq

Dnsmasq实际是一个本地DNS工具，原理是通过劫持本地的DNS请求，统一做转发处理。也可以对外暴露53接口(或者5353接口，然后通过iptables做转发)，以供其它VPS访问。

安装

```bash
apt-get install dnsmasq
```

在dnsmasq的配置文件( /etc/dnsmasq.conf )中加入指定配置文件夹

```bash
echo conf-dir=/etc/dnsmasq.d >> /etc/dnsmasq.conf
```

进入/etc/dnsmasq.d/ 新建一个netflix.conf

```bash
cd /etc/dnsmasq.d/ && vi netflix.conf
```

将以下\<IP\_ADDR>替换成当前vps的IP(IPv6亦可)

```bash
domain-needed
bogus-priv
no-resolv
no-poll
all-servers
server=8.8.8.8
server=1.1.1.1
server=208.67.222.222
server=2001:4860:4860::8888
server=2001:4860:4860::8844
cache-size=2048
local-ttl=60
interface=*
address=/akadns.net/<IP_ADDR>
address=/akam.net/<IP_ADDR>
address=/akamai.com/<IP_ADDR>
address=/akamai.net/<IP_ADDR>
address=/akamaiedge.net/<IP_ADDR>
address=/akamaihd.net/<IP_ADDR>
address=/akamaistream.net/<IP_ADDR>
address=/akamaitech.net/<IP_ADDR>
address=/akamaitechnologies.com/<IP_ADDR>
address=/akamaitechnologies.fr/<IP_ADDR>
address=/akamaized.net/<IP_ADDR>
address=/edgekey.net/<IP_ADDR>
address=/edgesuite.net/<IP_ADDR>
address=/srip.net/<IP_ADDR>
address=/footprint.net/<IP_ADDR>
address=/level3.net/<IP_ADDR>
address=/llnwd.net/<IP_ADDR>
address=/edgecastcdn.net/<IP_ADDR>
address=/cloudfront.net/<IP_ADDR>
address=/hulu.com/<IP_ADDR>
address=/huluim.com/<IP_ADDR>
address=/hbonow.com/<IP_ADDR>
address=/hbogo.com/<IP_ADDR>
address=/hbo.com/<IP_ADDR>
address=/amazon.com/<IP_ADDR>
address=/amazon.co.uk/<IP_ADDR>
address=/amazonvideo.com/<IP_ADDR>
address=/crackle.com/<IP_ADDR>
address=/pandora.com/<IP_ADDR>
address=/vudu.com/<IP_ADDR>
address=/blinkbox.com/<IP_ADDR>
address=/abc.com/<IP_ADDR>
address=/fox.com/<IP_ADDR>
address=/theplatform.com/<IP_ADDR>
address=/nbc.com/<IP_ADDR>
address=/nbcuni.com/<IP_ADDR>
address=/ip2location.com/<IP_ADDR>
address=/pbs.org/<IP_ADDR>
address=/warnerbros.com/<IP_ADDR>
address=/southpark.cc.com/<IP_ADDR>
address=/cbs.com/<IP_ADDR>
address=/brightcove.com/<IP_ADDR>
address=/cwtv.com/<IP_ADDR>
address=/spike.com/<IP_ADDR>
address=/go.com/<IP_ADDR>
address=/mtv.com/<IP_ADDR>
address=/mtvnservices.com/<IP_ADDR>
address=/playstation.net/<IP_ADDR>
address=/uplynk.com/<IP_ADDR>
address=/maxmind.com/<IP_ADDR>
address=/xboxlive.com/<IP_ADDR>
address=/lovefilm.com/<IP_ADDR>
address=/turner.com/<IP_ADDR>
address=/amctv.com/<IP_ADDR>
address=/sho.com/<IP_ADDR>
address=/mog.com/<IP_ADDR>
address=/wdtvlive.com/<IP_ADDR>
address=/beinsportsconnect.tv/<IP_ADDR>
address=/beinsportsconnect.net/<IP_ADDR>
address=/fig.bbc.co.uk/<IP_ADDR>
address=/open.live.bbc.co.uk/<IP_ADDR>
address=/sa.bbc.co.uk/<IP_ADDR>
address=/www.bbc.co.uk/<IP_ADDR>
address=/crunchyroll.com/<IP_ADDR>
address=/ifconfig.co/<IP_ADDR>
address=/omtrdc.net/<IP_ADDR>
address=/sling.com/<IP_ADDR>
address=/movetv.com/<IP_ADDR>
address=/happyon.jp/<IP_ADDR>
address=/abema.tv/<IP_ADDR>
address=/hulu.jp/<IP_ADDR>
address=/optus.com.au/<IP_ADDR>
address=/optusnet.com.au/<IP_ADDR>
address=/gamer.com.tw/<IP_ADDR>
address=/bahamut.com.tw/<IP_ADDR>
address=/hinet.net/<IP_ADDR>
address=/netflix.ca/<IP_ADDR>
address=/netflix.com/<IP_ADDR>
address=/netflix.net/<IP_ADDR>
address=/netflixinvestor.com/<IP_ADDR>
address=/netflixtechblog.com/<IP_ADDR>
address=/nflxext.com/<IP_ADDR>
address=/nflximg.com/<IP_ADDR>
address=/nflximg.net/<IP_ADDR>
address=/nflxsearch.net/<IP_ADDR>
address=/nflxso.net/<IP_ADDR>
address=/nflxvideo.net/<IP_ADDR>
address=/netflixdnstest0.com/<IP_ADDR>
address=/netflixdnstest1.com/<IP_ADDR>
address=/netflixdnstest2.com/<IP_ADDR>
address=/netflixdnstest3.com/<IP_ADDR>
address=/netflixdnstest4.com/<IP_ADDR>
address=/netflixdnstest5.com/<IP_ADDR>
address=/netflixdnstest6.com/<IP_ADDR>
address=/netflixdnstest7.com/<IP_ADDR>
address=/netflixdnstest8.com/<IP_ADDR>
address=/netflixdnstest9.com/<IP_ADDR>
address=/api.fast.com/<IP_ADDR>
address=/netflix.com.edgesuite.net/<IP_ADDR>
address=/disney.asia/<IP_ADDR>
address=/disney.be/<IP_ADDR>
address=/disney.bg/<IP_ADDR>
address=/disney.ca/<IP_ADDR>
address=/disney.ch/<IP_ADDR>
address=/disney.co.il/<IP_ADDR>
address=/disney.co.jp/<IP_ADDR>
address=/disney.co.kr/<IP_ADDR>
address=/disney.co.th/<IP_ADDR>
address=/disney.co.uk/<IP_ADDR>
address=/disney.co.za/<IP_ADDR>
address=/disney.com/<IP_ADDR>
address=/disney.com.au/<IP_ADDR>
address=/disney.com.br/<IP_ADDR>
address=/disney.com.hk/<IP_ADDR>
address=/disney.com.tw/<IP_ADDR>
address=/disney.cz/<IP_ADDR>
address=/disney.de/<IP_ADDR>
address=/disney.dk/<IP_ADDR>
address=/disney.es/<IP_ADDR>
address=/disney.fi/<IP_ADDR>
address=/disney.fr/<IP_ADDR>
address=/disney.gr/<IP_ADDR>
address=/disney.hu/<IP_ADDR>
address=/disney.id/<IP_ADDR>
address=/disney.in/<IP_ADDR>
address=/disney.io/<IP_ADDR>
address=/disney.it/<IP_ADDR>
address=/disney.my/<IP_ADDR>
address=/disney.nl/<IP_ADDR>
address=/disney.no/<IP_ADDR>
address=/disney.ph/<IP_ADDR>
address=/disney.pl/<IP_ADDR>
address=/disney.pt/<IP_ADDR>
address=/disney.ro/<IP_ADDR>
address=/disney.ru/<IP_ADDR>
address=/disney.se/<IP_ADDR>
address=/disney.sg/<IP_ADDR>
address=/20thcenturystudios.com.au/<IP_ADDR>
address=/20thcenturystudios.com.br/<IP_ADDR>
address=/20thcenturystudios.jp/<IP_ADDR>
address=/adventuresbydisney.com/<IP_ADDR>
address=/babble.com/<IP_ADDR>
address=/babyzone.com/<IP_ADDR>
address=/beautyandthebeastmusical.co.uk/<IP_ADDR>
address=/dilcdn.com/<IP_ADDR>
address=/disney-asia.com/<IP_ADDR>
address=/disney-discount.com/<IP_ADDR>
address=/disney-plus.net/<IP_ADDR>
address=/disney-studio.com/<IP_ADDR>
address=/disney-studio.net/<IP_ADDR>
address=/disneyadsales.com/<IP_ADDR>
address=/disneyarena.com/<IP_ADDR>
address=/disneyaulani.com/<IP_ADDR>
address=/disneybaby.com/<IP_ADDR>
address=/disneycareers.com/<IP_ADDR>
address=/disneychannelonstage.com/<IP_ADDR>
address=/disneychannelroadtrip.com/<IP_ADDR>
address=/disneycruisebrasil.com/<IP_ADDR>
address=/disneyenconcert.com/<IP_ADDR>
address=/disneyiejobs.com/<IP_ADDR>
address=/disneyinflight.com/<IP_ADDR>
address=/disneyinternational.com/<IP_ADDR>
address=/disneyinternationalhd.com/<IP_ADDR>
address=/disneyjunior.com/<IP_ADDR>
address=/disneyjuniortreataday.com/<IP_ADDR>
address=/disneylatino.com/<IP_ADDR>
address=/disneymagicmoments.co.il/<IP_ADDR>
address=/disneymagicmoments.co.uk/<IP_ADDR>
address=/disneymagicmoments.co.za/<IP_ADDR>
address=/disneymagicmoments.de/<IP_ADDR>
address=/disneymagicmoments.es/<IP_ADDR>
address=/disneymagicmoments.fr/<IP_ADDR>
address=/disneymagicmoments.gen.tr/<IP_ADDR>
address=/disneymagicmoments.gr/<IP_ADDR>
address=/disneymagicmoments.it/<IP_ADDR>
address=/disneymagicmoments.pl/<IP_ADDR>
address=/disneymagicmomentsme.com/<IP_ADDR>
address=/disneyme.com/<IP_ADDR>
address=/disneymeetingsandevents.com/<IP_ADDR>
address=/disneymovieinsiders.com/<IP_ADDR>
address=/disneymusicpromotion.com/<IP_ADDR>
address=/disneynewseries.com/<IP_ADDR>
address=/disneynow.com/<IP_ADDR>
address=/disneypeoplesurveys.com/<IP_ADDR>
address=/disneyplus.com/<IP_ADDR>
address=/disneyredirects.com/<IP_ADDR>
address=/disneysrivieraresort.com/<IP_ADDR>
address=/disneystore.com/<IP_ADDR>
address=/disneystreaming.com/<IP_ADDR>
address=/disneysubscription.com/<IP_ADDR>
address=/disneytickets.co.uk/<IP_ADDR>
address=/disneyturkiye.com.tr/<IP_ADDR>
address=/disneytvajobs.com/<IP_ADDR>
address=/disneyworld-go.com/<IP_ADDR>
address=/dssott.com/<IP_ADDR>
address=/go-disneyworldgo.com/<IP_ADDR>
address=/go.com/<IP_ADDR>
address=/mickey.tv/<IP_ADDR>
address=/moviesanywhere.com/<IP_ADDR>
address=/nomadlandmovie.ch/<IP_ADDR>
address=/playmation.com/<IP_ADDR>
address=/shopdisney.com/<IP_ADDR>
address=/shops-disney.com/<IP_ADDR>
address=/sorcerersarena.com/<IP_ADDR>
address=/spaindisney.com/<IP_ADDR>
address=/star-brasil.com/<IP_ADDR>
address=/star-latam.com/<IP_ADDR>
address=/starwars.com/<IP_ADDR>
address=/starwarsgalacticstarcruiser.com/<IP_ADDR>
address=/starwarskids.com/<IP_ADDR>
address=/streamingdisney.net/<IP_ADDR>
address=/thestationbymaker.com/<IP_ADDR>
address=/thisispolaris.com/<IP_ADDR>
address=/watchdisneyfe.com/<IP_ADDR>
address=/bamgrid.com/<IP_ADDR>
address=/go-mpulse.net/<IP_ADDR>
address=/newrelic.com/<IP_ADDR>
address=/nr-data.net/<IP_ADDR>
address=/akstat.io/<IP_ADDR>
address=/disney.api.edge.bamgrid.com/<IP_ADDR>
address=/global.edge.bamgrid.com/<IP_ADDR>
address=/edge.bamgrid.com/<IP_ADDR>
```

重启dnsmasq

```bash
systemctl restart dnsmasq
```

保证启动服务

```bash
systemctl enable dnsmasq
```

通过dig工具查看解析

```bash
dig netflix.com ANY @127.0.0.1
```

### **SNIProxy**

[SNIProxy github 项目地址](https://github.com/dlundquist/sniproxy)

必须先apt update

```bash
sudo apt update
```

安装依赖项

```bash
sudo apt-get install autotools-dev cdbs debhelper dh-autoreconf dpkg-dev gettext libev-dev libpcre3-dev libudns-dev pkg-config fakeroot devscripts
```

切换到根目录，克隆sniproxy项目

```bash
cd ~ && mkdir sniproxydown; cd sniproxydown; git clone <https://github.com/dlundquist/sniproxy.git>
```

切换到sniproxy目录

```bash
cd sniproxy
```

创建dpkg包

```bash
./autogen.sh && dpkg-buildpackage
```

运行完之后，会在上层目录产生几个包文件，安装对应的包

```bash
cd .. && sudo dpkg -i sniproxy_0.6.0+git.10.g822bb80_amd64.deb
```

安装完毕之后，会有一个conf文件，编辑

```bash
vi /etc/sniproxy.conf
```

可参照如下配置进行对应修改

```bash
user daemon

pidfile /var/run/sniproxy.pid

error_log {
    syslog daemon
    priority notice
}

resolver {
    # mode ipv6_first
    # mode ipv6_only
    # mode ipv4_first
    mode ipv4_only
}

access_log {
    filename /tmp/sniproxy-access.log
}

listen 80 {
    proto http
    table http_hosts
    # Fallback backend server to use if we can not parse the client request
    # fallback localhost:8080

    access_log {
        filename /var/log/sniproxy/http_access.log
        priority notice
    }
}

listen 443 {
    proto tls
    table https_hosts

    access_log {
        filename /var/log/sniproxy/https_access.log
        priority notice
    }
}

table http_hosts {
    .* *:80
}

table https_hosts {
    .* *:443
}
```

设置开机自动启动sniproxy

参考： [SNI Proxy Tutorial · GitHub](https://gist.github.com/pjamar/6812af8354746f9bffd0)

```bash
vi /etc/default/sniproxy
```

设置：

```bash
ENABLED=1
```

保存，退出，重启

\*\*\*\*\*\*\*\*\*\* 以下部分仅供参考 \*\*\*\*\*\*\*\*\*\*

**( 这一部分仅供参考，原因是sniproxy的包所放服务是在/etc/init.d下，启动并非由systemctl来控制，并且使用systemctl控制可能会有问题。原包下sniproxy也会自动读取 /etc/sniproxy.conf )**

注意，目前这个包会自动在 /etc/init.d 放入 sniproxy，我们把它移除出去，手动重建一个

```bash
# 先停止并且disable
systemctl stop sniproxy
systemctl disable sniproxy

# 移除
cd /etc/init.d
mv sniproxy /root/sniproxy
```

手动新建一个mysni.service

```bash
cd /etc/systemd/system/ && vi mysni.service
```

添加具体服务启动内容指向 /etc/sniproxy.conf

```bash
[Unit]
Description=SNIProxy server
After=network.target

[Service]
Type=simple
ExecStart=/usr/sbin/sniproxy -c /etc/sniproxy.conf
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

重启daemon

```bash
systemctl daemon-reload
```

尝试启动sniproxy

```bash
systemctl restart mysni
```

查看sniproxy状态

```bash
# 是否启动
systemctl status mysni 

# 是否有进程
lsof | grep sniproxy
```

如果成功了，则继续将mysni设为enable

```bash
systemctl enable mysni
```

\*\*\*\*\*\*\*\*\*\* 以上部分仅供参考 \*\*\*\*\*\*\*\*\*\*

sniproxy安装好之后会自动启动，查看是否启动成功

```bash
lsof | grep sniproxy
```

如果没有成功，需要查看具体的log

```bash
# 进入系统日志
cd /var/log
# 查看报错信息
cat syslog
```

可能会发现sniproxy没有成功是因为有其它进程占用了80端口，所以无法listen

```bash
# 查看是谁占用了80端口
netstat -apn | grep 80
```

发现是apache2服务占用了80端口，那就关闭它

```bash
systemctl stop apache2
systemctl disable apache2
```

然后尝试重启sniproxy

```bash
sniproxy stop
sniproxy start
```

至此应该已经可以使用了

注意，如果访问站点有问题，需要看下mode是否设置了 ipv4\_only

**IMPORTANT 重要**

sniproxy打开之后, 一定要设置iptables, 防止被别人利用导致流量超标

安装使用iptables-persistent

```bash
sudo apt-get install iptables-persistent

sudo netfilter-persistent save

sudo netfilter-persistent reload
```

关闭所有80/443的访问, 只留允许的IP

```bash
# 丢弃所有80/443的访问(不回应对方的请求)
iptables -I INPUT -p tcp --dport 80 -j DROP
iptables -I INPUT -p tcp --dport 443 -j DROP

# 阻止所有80/443的访问(告诉对方被拒绝)
iptables -I INPUT -p tcp --dport 80 -j REJECT
iptables -I INPUT -p tcp --dport 443 -j REJECT

# 仅允许IP 11.22.33.44 的80/443的访问
iptables -I INPUT -s 11.22.33.44 -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -s 11.22.33.44 -p tcp --dport 443 -j ACCEPT

# IPV6 阻止所有IPV6 80/443 的访问
ip6tables -I INPUT -p tcp --dport 80 -j DROP
ip6tables -I INPUT -p tcp --dport 443 -j DROP
ip6tables -I INPUT -p tcp --dport 80 -j REJECT
ip6tables -I INPUT -p tcp --dport 443 -j REJECT

# 保存修改
sudo netfilter-persistent save
# 重载规则
sudo netfilter-persistent reload
```

**各种Tips**

如果修改了 /etc/sniproxy.conf 不起效，或者明明代理dns了但访问返回 _ERR\_SSL\_VERSION\_OR\_CIPHER\_MISMATCH_ 错误，那就是sniproxy服务没有重启成功！！

首先，关闭sniproxy服务

```bash
systemctl stop sniproxy
```

查看sniproxy是否真的被关闭，需要查看进程

```bash
lsof | grep sniproxy
```

如果还有sniproxy在运行，直接用kill 来强制关闭进程(参数 -9)

```bash
kill -9 <process id>
```

这时候再重启sniproxy

```bash
systemctl restart sniproxy
```

另外查看一下sniproxy是否加载了对应的conf文件

```bash
systemctl status sniproxy
```

如果此时没有出现 /etc/sniproxy.conf，则需要手动指定conf文件

```bash
sniproxy -c /etc/sniproxy.conf
```

最后，再确认sniproxy进程是否 运行

```bash
lsof | grep sniproxy
```

如果此时没有sniproxy进程，说明 conf 文件配置错误，这时就需要去查看并修改conf文件

查看端口占用

```bash
netstat -apn | grep 8080
```

