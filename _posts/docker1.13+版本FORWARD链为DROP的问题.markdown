---
layout:     post
title:      "docker 1.13+ FORWARD链为DROP的问题"
subtitle:   "Docker 安全漏洞"
date:       2017-03-16
author:     "Antony"
header-img: "img/docker.jpg"
tags:
    - docker
    - docker-forward-drop
---
### 本地网络容器访问漏洞
这个问题出现在docker的[issue](https://github.com/docker/docker/issues/14041)中

    

其主要影响：允许与docker主机位于同一网络上的任何人访问在该主机上运行的容器，而不是只访问暴露的端口。

  

当docker启动时，它启用net.ipv4.ip_forward而不改变iptables FORWARD链默认策略DROP。这意味着与docker主机位于同一网络上的另一台机器可以向其路由表添加路由，并直接寻址在该docker主机上运行的任何容器。

   

**例如，如果docker0子网是172.17.0.0/16（默认子网），并且docker主机的IP地址是192.168.0.10，则从网络上的另一个主机运行：**

```

ip route add 172.17.0.0/16 via 192.168.0.10

nmap 172.17.0.0/16

```

上面将扫描主机上运行的容器，并报告找到的IP地址和运行的服务。

要解决这个问题，docker需要将FORWARD策略设置为DROP启用net.ipv4.ip_forwardsysctl参数。

### 带来的其他问题

带来的问题主要出现在docker 配合kubernetes使用时，kubernetes的网络使用flannel，flannel可以使不同主机间的容器通信。如下图

![docker-forward-drop](http://obbogqhb1.bkt.clouddn.com/docker-bug.png)

但是带来的问题是：一旦DROP后，不同主机间就不能正常通信了，为了保证其能正常通信，可以选择三种方案。

#### 1.将FORWARD改为ACCEPT

```

$ iptables -P FORWARD ACCEPT

这种方式不适合重启，一重启就需要修改

```

#### 2.仅允许flannel网段

```

$ iptables -I FORWARD -s 172.1.0.0/16 -j ACCEPT

```

#### 3.在/usr/lib/systemd/system/docker.service中添加

```

[Service] #在这下面添加

ExecStartPost=/sbin/iptables -I FORWARD -s 0.0.0.0/0 -j ACCEPT

```
