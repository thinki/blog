title: ubuntu搭建pppoe server测试环境
date: 2015-06-30 12:54:26
tags: PPPoE
---

最近需要Port PPPoE dial up软件，但是又没有现成的ISP提供的PPPoE端口，索性就自己搭一个吧，其实过程也不复杂，不过中途遇到了一些问题，在此记录下来，以备日后温故而知新。

##硬件要求
首先需要的是两张以太网卡，一般主板会自带一张，另一张的话自己去网上买个PCIE Ethernet Adapter，蟹厂的东西还是比较便宜的，Ubuntu上一般也是自带了驱动。

##软件要求
###1. Ubuntu上装软件了：
```bash
sudo apt-get install ppp
```
这里讲一下关系，PPPoE全程是PPP over Ethernet，也就是在PPP协议的基础上实现的，所以需要安装ppp包，然后这里可以安装一个PPPoE的client，方便配置脚本等等。
###2. 下载rp-pppoe并解压安装，注意sudo权限
```bash
wget http://www.roaringpenguin.com/files/download/rp-pppoe-3.10.tar.gz
tar xvf rp-pppoe-3.10.tar.gz
cd rp-pppoe-3.10/src
./configure
make
sudo make install
```
###3. 编辑PPPoE server配置文件
```bash
sudo vim /etc/ppp/pppoe-server-options
```
```bash
require-chap
lcp-echo-interval 10
lcp-echo-failure 2
ms-dns 8.8.8.8  #dns需要修改，国内开源dns已被封
defaultroute
```
如果加上login选项，那么[登陆的用户名必需和linux系统下的一个用户名相同](http://blog.csdn.net/pdcxs007/article/details/44599885)
###4. 添加拨号账号密码
```bash
sudo vim /etc/ppp/chap-secrets
```

```
# Secrets for authentication using CHAP
# client server secret IP addresses
 
#USERNAME       SERVER		PASSWORD			CLIENT IP ADDRESS
"cristanhuza"	*"		"My_s3cret"			*
"friend1"	*		"My_friend"			192.168.1.2
```

###5. 开启PPPoE Server
这里假定eth0是默认的上网口，eth1是PPPoE实验端口
```
pppoe-server -I eth1 -L 192.168.0.20 -R 192.168.0.30 -N 10
```
-I eth1 指定pppoe服务器在那个网卡接口监听连接请求

-L 192.168.0.20 指定pppoe服务器的ip地址。（注意：此IP地址不是网卡的IP地址，而是PPPOE服务器的虚拟IP）

-R 192.168.0.30 pppoe服务器分配给客户端的IP地址，从192.168.0.30开始，递增

-N 10 指定最多可以连接pppoe服务器的客户端数量

###6. 使能packet forwarding
注意这里需要先切换root用户才能够操作成功
```bash
sudo -i
echo 1 > /proc/sys/net/ipv4/ip_forward
```

###7. iptables添加NAT规则
```bash
sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
```

###Windows NT拨号
到这里基本上算是配置完成，在windows端配置PPPoE这里就不再阐述，值得注意的是由于PPPoE属于PPP点到点协议，因此分配到的netmask是255.255.255.255，并且也不需要配置网络端口的IP，PPPoE拨号成功后会另外分配IP到宽带拨号适配器上

参考链接：
1. [Ubuntu上架设PPPoE Server](http://blog.csdn.net/linweig/article/details/5481355)
2. [PPPoE Server – How To Do It Yourself](http://www.howtodoityourself.org/pppoe-server-how-to-do-it-yourself.html)
3. [Linux下搭建 PPPoE Server 问题总结](http://blog.csdn.net/pdcxs007/article/details/44599885)
