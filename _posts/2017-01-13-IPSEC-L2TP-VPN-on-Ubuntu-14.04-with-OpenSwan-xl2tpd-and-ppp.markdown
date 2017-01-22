---
layout:     post
title:      "IPSEC-L2TP-VPN-on-Ubuntu-14.04-with-OpenSwan-xl2tpd-and-ppp"
subtitle:   "L2TP/IPSEC VPN"
date:       2017-01-13
author:     "Antony"
header-img: "img/vpn.jpg"
tags:
    - vpn
    - l2tp/ipsec
    - l2tp
    - ipsec

---
### VPN原理

http://www.cisco.com/support/zh/105/IPSECpart1.shtml#glossary

http://www.china-ccie.com/doc/vpn/vpn.html

### Install ppp openswan and xl2tpd

```

$ sudo apt-get install openswan xl2tpd ppp lsof

```

### Firewall and sysctl

#### 配置一条防火墙语句

```

iptables -t nat -A POSTROUTING -s %SERVERIP% -o eth0 -j MASQUERADE

```

#### 启用内核转发和禁用ICP redirects

```

echo "net.ipv4.ip_forward = 1" |  tee -a /etc/sysctl.conf

echo "net.ipv4.conf.all.accept_redirects = 0" |  tee -a /etc/sysctl.conf

echo "net.ipv4.conf.all.send_redirects = 0" |  tee -a /etc/sysctl.conf

echo "net.ipv4.conf.default.rp_filter = 0" |  tee -a /etc/sysctl.conf

echo "net.ipv4.conf.default.accept_source_route = 0" |  tee -a /etc/sysctl.conf

echo "net.ipv4.conf.default.send_redirects = 0" |  tee -a /etc/sysctl.conf

echo "net.ipv4.icmp_ignore_bogus_error_responses = 1" |  tee -a /etc/sysctl.conf

```

#### 设置如下参数到其他网络接口

```

$ for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done
$ sysctl -p

```

#### 设置重启后执行`/etc/rc.local`

```

$ sudo vim /etc/rc.local

for vpn in /proc/sys/net/ipv4/conf/*; do echo 0 > $vpn/accept_redirects; echo 0 > $vpn/send_redirects; done



iptables -t nat -A POSTROUTING -s %SERVERIP% -o eth0 -j MASQUERADE

```

### 配置 Openswan(IPSEC)

使用你的编辑器打开ipsec的配置文件

```

/etc/ipsec.conf

```

配置如下

```

config setup

      nat_traversal=yes

      virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12,%v4:!10.152.2.0/24

      # 这里包含的网络地址允许配置为远程客户端所在的子网。换句话说，

      # 这些地址范围应该是你的NAT路由器后面的客户端的地址。

      oe=off

      protostack=netkey



conn L2TP-PSK-NAT

      rightsubnet=vhost:%priv

      also=L2TP-PSK-noNAT



conn L2TP-PSK-noNAT

      authby=secret

      pfs=no

      auto=add

      keyingtries=3

      rekey=no

      # Apple 的 iOS 不会发送 delete 提醒，

      # 所以我们需要通过死亡对端（dead peer）检测来识别断掉的客户端

      dpddelay=30

      dpdtimeout=120

      dpdaction=clear

      # 设置 ikelifetime 和 keylife 和 Windows 的默认设置一致

      ikelifetime=8h

      keylife=1h

      type=transport

      # 替换 IP 地址为你的本地IP （一般是，私有地址、NAT内的地址）

      left=10.100.1.217

      # 用于升级过的 Windows 2000/XP 客户端

      leftprotoport=17/1701

      # 要支持老的客户端，需要设置 leftprotoport=17/%any

      right=%any

      rightprotoport=17/%any

      # 强制所有连接都NAT，因为 iOS

      forceencaps=yes

```

通过如下命令可以获取你本地的网络ip(当然，是外网ip，如果你想使用内网ip，请忽略)

```

curl ifconfig.me

# 或者

curl http://ip.mtak.nl

```

#### 配置共享密钥

`/etc/ipsec.secrets` file,确保其长度够

```

%SERVER_IP%  %any  : PSK "BC2BCKALFJKAJFKAMCJKFKSJKNCMZJKAK443"

```

如果你想生成一个random key，使用openssl命令

```

$ sudo openssl rand -hex 30

39651bbc2c459109410b3b28e3507d27799feb5e8e0dc25532d29274abb7

```

#### 校验ipsec设置

```

ipsec verify

Verifying installed system and configuration files



Version check and ipsec on-path                   	[OK]

Libreswan 3.18 (netkey) on 3.13.0-101-generic

Checking for IPsec support in kernel              	[OK]

 NETKEY: Testing XFRM related proc values

         ICMP default/send_redirects              	[OK]

         ICMP default/accept_redirects            	[OK]

         XFRM larval drop                         	[OK]

Pluto ipsec.conf syntax                           	[OK]

Hardware random device                            	[N/A]

Two or more interfaces found, checking IP forwarding	[OK]

Checking rp_filter                                	[OK]

Checking that pluto is running                    	[OK]

 Pluto listening for IKE on udp 500               	[OK]

 Pluto listening for IKE/NAT-T on udp 4500        	[OK]

 Pluto ipsec.secret syntax                        	[OK]

Checking 'ip' command                             	[OK]

Checking 'iptables' command                       	[OK]

Checking 'prelink' command does not interfere with FIPSChecking for obsolete ipsec.conf options          	[OK]

Opportunistic Encryption                          	[DISABLED]

```

### 配置 xl2tpd

`/etc/xl2tpd/xl2tpd.conf` 配置文件

#### 修改其配置

```

[global]

ipsec saref = no



[lns default]

ip range = 10.0.1.230-10.0.1.240

local ip = 10.0.1.217

require chap = yes

refuse pap = yes

require authentication = yes

ppp debug = yes

pppoptfile = /etc/ppp/options.xl2tpd

length bit = yes

```

- ip range = 主要提供给连接的客户端使用

- local ip = vpn server ip

- refuse pap = refuse pap认证

- ppp debug = yes 测试的时候使用，不要再生产使用

其他认证方式：https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_with_Ubuntu_14.04.html

### 配置ppp

```

$ vim /etc/ppp/options.xl2tpd

refuse-mschap-v2

refuse-mschap

ms-dns 114.114.114.114

ms-dns 202.106.0.20

asyncmap 0

auth

crtscts

idle 1800

mtu 1200

mru 1200

lock

hide-password

local

#debug

name l2tpd

proxyarp

lcp-echo-interval 30

lcp-echo-failure 4

```

- ms-dns: 提供给客户端的dns地址

- proxyarp：添加一个加密给系统ARP表(Add an entry to this systems ARP [Address Resolution Protocol] table with the IP address of the peer and the Ethernet address of this system. This will have the effect of making the peer appear to other systems to be on the local ethernet.)

- name l2tpd: 用于ppp认证文件给l2tpd

#### 添加一个用户

```

$ sudo vim /etc/ppp/chap-secrets

"test" l2tpd "test@123..$$_@" *

```

### 重启服务并测试

```


/etc/init.d/ipsec restart 

/etc/init.d/xl2tpd restart

```

日志存储路径

```

/var/log/syslog

/var/log/auth.log

```

### 其他平台配置

- [CentOS 7, Scientific Linux 7 or Red Hat Enterprise Linux 7 (IKEv2,no L2TP)](https://raymii.org/s/tutorials/IPSEC_vpn_with_CentOS_7.html)

- [CentOS 6, Scientific Linux 6 or Red Hat Enterprise Linux 6](https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_on_CentOS_-_Red_Hat_Enterprise_Linux_or_Scientific_-_Linux_6.html)

- [Raspberry Pi with Arch Linux ARM](https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_on_a_Raspberry_Pi_with_Arch_Linux.html)

- [Ubuntu 16.04, (IKEv2,no L2TP)](https://raymii.org/s/tutorials/IPSEC_vpn_with_Ubuntu_16.04.html)

- [Ubuntu 15.10, (IKEv2,no L2TP)](https://raymii.org/s/tutorials/IPSEC_vpn_with_Ubuntu_15.10.html)

- [Ubuntu 15.04, (IKEv2,no L2TP)](https://raymii.org/s/tutorials/IPSEC_vpn_with_Ubuntu_15.04.html)

- [Ubuntu 13.10](https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_with_Ubuntu_13.10.html)

- [Ubuntu 13.04](https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_with_Ubuntu_13.04.html)

- [Ubuntu 12.10](https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_with_Ubuntu_12.10.html)

- [Ubuntu 12.04 LTS](https://raymii.org/s/tutorials/IPSEC_L2TP_vpn_with_Ubuntu_12.04.html)




>作者：`Antony` WX&QQ：`1257465991`
Q/A：如有问题请慷慨提出
