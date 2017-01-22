---
layout:     post
title:      "添加一个用户sudo权限的使用密钥登录的用户"
subtitle:   "ALL Permission Sudo User "
date:       2017-01-21
author:     "Antony"
header-img: "img/useradd.jpg"
tags:
    - shell
---
#### $1为用户名
#### $2为用户公钥
```
#!/bin/bash
#添加证书账户，并且将对应用户加入sudo
# $1 为用户名 $2 为用户公钥
[[ -z $3 ]] && useradd $1 || useradd -u $3 $1
mkdir /home/$1/.ssh
echo "$2" >/home/$1/.ssh/authorized_keys
chown -R $1.$1 /home/$1/.ssh/
chmod 600 /home/$1/.ssh/authorized_keys
chmod 700 /home/$1/.ssh/
sed -i '/^root.*/a\'$1' ALL=(ALL) NOPASSWD:ALL' /etc/sudoers
```
