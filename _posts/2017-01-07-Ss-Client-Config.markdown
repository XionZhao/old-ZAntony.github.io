---
layout:     post
title:      "Ss-Client-Config"
subtitle:   "Ss-Client-Config"
date:       2017-01-10
author:     "Antony"
header-img: "img/post-bg-js-module.jpg"
tags:
    - FastDFS
    - 分布式存储
---
### Mac local configure

```

$ sudo pip install shadowsocks

$ sudo vim /etc/shadowsock-cient.json

{

    "server":"SERVER_IP",

    "server_port":PORT,

    "password":"PASSWORD",

    "timeout":600,

    "method":"aes-256-cfb",

    "fast_open": false

}

$ sudo sslocal -c /etc/shadowsock-client.json -d start 

```

### Windows local configure

待补充。。。。

### SwitchyOmega 配置

### 1、下载chrom浏览器插件

https://chrome.google.com/webstore/detail/proxy-switchyomega/padekgcemlokbadohgkifijomclgjgif?hl=zh-CN

### 2、导入我的备份文件

先下载我的备份：http://zantony.top:8088/antony.bak

![](/img/ss/1.png)

### 3、配置情景

![](/img/ss/2.png)

### 4、设置为自动切换
![](/img/ss/3.png)
![](/img/ss/4.png)

### 5、畅游自由网络的海洋

![](/img/ss/5.png)
![](/img/ss/6.png)
![](/img/ss/7.png)






