---
layout:     post
title:      "COW (Climb Over the Wall) Proxy"
subtitle:   "穿墙的 HTTP 代理服务器"
date:       2017-02-20
author:     "Antony"
header-img: "img/yahoo.jpg"
tags:
    - ss
    - cow
---
### COW简介

> `COW` 是一个简化穿墙的 `HTTP 代理服务器`。它能自动检测被墙网站，仅对这些网站使用`二级代理`。COW对外提供一个地址和端口，用户只需将这个地址配置好，便可以使用。COW的二级代理用来访问海外的server，支持`sock5`、`shadowsocks`、`cow`(采用shadowsock协议)三种方式. [Github地址](https://github.com/cyfdecyf/cow)

### 功能

COW 的设计目标是自动化，理想情况下用户无需关心哪些网站无法访问，可直连网站也不会因为使用二级代理而降低访问速度。

- 作为 HTTP 代理，可提供给移动设备使用；若部署在国内服务器上，可作为 APN 代理

- 支持 HTTP, SOCKS5, shadowsocks 和 cow 自身作为二级代理

- 可使用多个二级代理，支持简单的负载均衡

- 自动检测网站是否被墙，仅对被墙网站使用二级代理

- 自动生成包含直连网站的 PAC，访问这些网站时可绕过 COW

- 内置常见可直连网站，如国内社交、视频、银行、电商等网站（可手工添加）

快速开始



### 一、安装

#### **1. OS X, Linux (x86, ARM)**: 执行以下命令（也可用于更新）

```

curl -L git.io/cow | bash



# 环境变量 `COW_INSTALLDIR` 可以指定安装的路径，若该环境变量不是目录则询问用户

```

#### **2.Windows**: 从 [release](https://github.com/cyfdecyf/cow/releases) 页面下载



#### **3.熟悉 Go 的用户可用 go get github.com/cyfdecyf/cow 从源码安装**

```

go get github.com/cyfdecyf/cow

```

### 二、配置



**编辑 ~/.cow/rc (Linux) 或 rc.txt (Windows)，简单的配置例子如下**

```



#开头的行是注释，会被忽略

# 本地 HTTP 代理地址

# 配置 HTTP 和 HTTPS 代理时请填入该地址

# 若配置代理时有对所有协议使用该代理的选项，且你不清楚此选项的含义，请勾选

# 或者在自动代理配置中填入 http://127.0.0.1:7777/pac

listen = http://127.0.0.1:7777

# SOCKS5 二级代理 用来连接socks5

proxy = socks5://127.0.0.1:1080

# HTTP 二级代理 用来连接其他的http代理

proxy = http://127.0.0.1:8080

proxy = http://user:password@127.0.0.1:8080

# shadowsocks 二级代理 用来连接ss代理

proxy = ss://aes-128-cfb:password@1.2.3.4:8388

# cow 二级代理 用来连接cow的二级代理

proxy = cow://aes-128-cfb:password@1.2.3.4:8388

```

**使用 cow 协议的二级代理需要在国外服务器上安装 COW，并使用如下配置：**

```

listen = cow://aes-128-cfb:password@0.0.0.0:8388

```

完成配置后启动 COW 并配置好代理即可使用。



### 三、启动COW

**配置文件在 Unix 系统上为 ~/.cow/rc，Windows 上为 COW 所在目录的 rc.txt 文件。 [样例配置](http://zantony.top/2017/02/20/Cow-rc-%E6%A0%B7%E4%BE%8B%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6/) 包含了所有选项以及详细的说明，建议下载然后修改。**

- **Unix 系统在命令行上执行 cow & (若 COW 不在 PATH 所在目录，请执行 ./cow &)**

```

cow -rc=rc  -v  -debug & # 可先开启debug模式测试

```

- **Windows**

 - 双击 cow-taskbar.exe，隐藏到托盘执行

 - 双击 cow-hide.exe，隐藏为后台程序执行



>以上两者都会启动 cow.exe

>PAC url 为 http://<listen address>/pac，也可将浏览器的 HTTP/HTTPS 代理设置为 listen address 使所有网站都通过 COW 访问。



**使用 PAC 可获得更好的性能，但若 PAC 中某网站从直连变成被封，浏览器会依然尝试直连。遇到这种情况可以暂时不使用 PAC 而总是走 HTTP 代理，让 COW 学习到新的被封网站。命令行选项可以覆盖部分配置文件中的选项、打开 debug/request/reply 日志，执行 cow -h 来获取更多信息。**

### 四、客户端配置

#### 1.Firefox配置

首选项-->高级-->网络-->设置
![firefox](http://obbogqhb1.bkt.clouddn.com/firefox.png)



#### 2.Chrom和mac配置

网络-->选择网络-->高级-->代理
![mac](http://obbogqhb1.bkt.clouddn.com/mac1.png)
![mac2](http://obbogqhb1.bkt.clouddn.com/mac2.png)

### 五、手动指定被墙和直连网站

一般情况下无需手工指定被墙和直连网站，该功能只是是为了处理特殊情况和性能优化。

配置文件所在目录下的 blocked 和 direct 可指定被墙和直连网站（direct 中的 host 会添加到 PAC）。 Windows 下文件名为 blocked.txt 和 direct.txt。

- 每行一个域名或者主机名（COW 会先检查主机名是否在列表中，再检查域名）

 - 二级域名如 google.com 相当于 *.google.com

 - com.hk, edu.cn 等二级域名下的三级域名，作为二级域名处理。如 google.com.hk 相当于 *.google.com.hk

 - 其他三级及以上域名/主机名做精确匹配，例如 plus.google.com



