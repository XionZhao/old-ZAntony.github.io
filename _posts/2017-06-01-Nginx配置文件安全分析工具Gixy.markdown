---
layout:     post
title:      "Nginx配置文件安全分析工具Gixy"
subtitle:   "Gixy is a tool to analyze Nginx configuration"
date:       2017-06-01
author:     "Antony"
header-img: "img/nginx-gixy.jpg"
tags:
    - nginx
    - nginx-conf
---
### Nginx配置文件安全分析工具Gixy
Gixy是一个分析Nginx配置的工具。Gixy的主要目的是防止安全的错误配置和自动化探伤。

目前支持Python版本2.7和3.5 +。

免责声明:Gixy只在GNU / Linux Gixy测试良好,其他的操作系统可能有问题。
#### Gixy的特性
- 找出服务器端请求伪造。
- 验证HTTP拆分。
- 验证referrer/origin问题。
- 验证是否正确通过add_header指令重新定义Response Headers。
- 验证请求的主机头是否伪造。
- 验证valid_referers是否为空。
- 验证是否存在多行主机头。

Github地址：[https://github.com/yandex/gixy](https://github.com/yandex/gixy)

#### Gixy安装
安装步骤很简单，直接使用pip安装即可
```
$ sudo pip install gixy
# 注意：目前支持Python版本2.7和3.5 +。
```
#### Gixy使用
Gixy默认检查/etc/nginx/nginx.conf文件

```
$ gixy
```

可以指定nginx配置文件所在位置

```
$ gixy /usr/local//nginx/nginx.conf

==================== Results ===================

Problem: [http_splitting] Possible HTTP-Splitting vulnerability.
Description: Using variables that can contain "\n" may lead to http injection.
Additional info: https://github.com/yandex/gixy/blob/master/docs/ru/plugins/httpsplitting.md
Reason: At least variable "$action" can contain "\n"
Pseudo config:
include /etc/nginx/sites/default.conf;

	server {

		location ~ /v1/((?<action>[^.]*)\.json)?$ {
			add_header X-Action $action;
		}
	}


==================== Summary ===================
Total issues:
    Unspecified: 0
    Low: 0
    Medium: 0
    High: 1
```

从结果查看出问题为`http_splitting`。原因是 $action 变量中可以含有换行符。这就是HTTP响应头拆分漏洞，通过CRLFZ注入实现攻击。

如果你要暂时忽略某类错误，可以使用--skip参数：

```
$ gixy --skips http_splitting /usr/local/nginx/nginx.conf

==================== Results ===================
No issues found.

==================== Summary ===================
Total issues:
    Unspecified: 0
    Low: 0
    Medium: 0
    High: 0
```
