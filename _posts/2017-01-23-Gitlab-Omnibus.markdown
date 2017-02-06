---
layout:     post
title:      "Gitlab-Omnibus"
subtitle:   "Gitlab-Omnibus 包安装方式"
date:       2017-02-04
author:     "Antony"
header-img: "img/gitlab.jpg"
tags:
    - Gitlab
---
### 一、Gitlab简介

![Gitlab1.png](http://obbogqhb1.bkt.clouddn.com/Gitlab1.png)

**组件说明**

- **前端**：Nginx，用于页面及Git tool走http或https协议

- **后端**：Gitlab服务，采用Ruby on Rails框架，通过unicorn实现后台服务及多进程

- **SSHD**：开启sshd服务，用于用户上传ssh key进行版本克隆及上传。注：用户上传的ssh key是保存到git账户中

- **数据库**：目前仅支持MySQL和PostgreSQL

- **Redis**：用于存储用户session和任务，任务包括新建仓库、发送邮件等等

- **Sidekiq**：Rails框架自带的，订阅redis中的任务并执行

**版本说明**

- **CE(GitLab Community Edition)**：社区版(源码安装方式) https://docs.gitlab.com/ce/README.html

- **OM(Omnibus GitLab)**：综合版(包安装方式) https://docs.gitlab.com/omnibus/README.html

- **EE(GitLab Enterprise Edition)**：企业版 https://docs.gitlab.com/ee/README.html

### 二、Gitlab Omnibus 安装和配置

**1. 安装和配置依赖环境**

```
sudo apt-get install curl openssh-server ca-certificates postfix
```

**2. 添加gitlab 包并安装**

```
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo apt-get install gitlab-ce
```

如果安装很慢，可以选择下载程序包后安装

```
https://packages.gitlab.com/gitlab/gitlab-ce/install
```

**3. 配置gitlab**

```
sudo gitlab-ctl reconfigure
```
**4.配置Nginx**

https://docs.gitlab.com/omnibus/settings/nginx.html

`配置https`

```
$ mkdir -p /etc/gitlab/ssl

$ chmod 700 /etc/gitlab/ssl

$ cp /etc/gitlab/ssl/* /etc/gitlab/trusted-certs/ # 注意：ssl下面的是证书文件

$ vim /etc/gitlab/gitlab.rb

external_url "https://git.meiqia.com"

nginx['redirect_http_to_https'] = true

nginx['ssl_certificate'] = "/etc/gitlab/ssl/meiqia.com.chain.crt"

nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/meiqia.com.key"

 nginx['proxy_set_headers'] = {

  "X-Forwarded-Proto" => "http",

  "CUSTOM_HEADER" => "VALUE"

 }

```

**5.配置LDAP**

参考配置：https://docs.gitlab.com/ce/administration/auth/ldap.html

```

gitlab_rails['ldap_enabled'] = true

gitlab_rails['ldap_servers'] = YAML.load <<-'EOS' # remember to close this block with 'EOS' below

  main: # 'main' is the GitLab 'provider ID' of this LDAP server

     label: 'meiqia-ldap'



     host: 'LDAP_HOST'

     port: 389

     uid: 'uid'

     bind_dn: 'cn=Directory Manager'

     password: 'PASSWORD'

     base: 'ou=People,dc=meiqia,dc=com'

     active_directory: false

     method: 'plain' # "tls" or "ssl" or "plain"

     user_filter: '(objectClass=meiqia-ldap)'

EOS

```

**6.配置EMAIL**

参考配置：https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/doc/settings/smtp.md

```

$ sudo vim /etc/gitlab/gitlab.rb

gitlab_rails['smtp_enable'] = true

gitlab_rails['smtp_address'] = "Your_EMAIL_SERVER_ADDRESS"

gitlab_rails['smtp_port'] = 25

gitlab_rails['smtp_user_name'] = "EMAIL_SERVER_USER"

gitlab_rails['smtp_password'] = "EMAIL_SERVER_PASSWORD"

gitlab_rails['smtp_authentication'] = "login"

gitlab_rails['gitlab_email_from'] = 'xxx@xx.com'

```

**7.重新加载配置**

```

$ sudo gitlab-ctl reconfigure

```

**Testing the SMTP configuration**

```

$ sudo gitlab-rails console

irb(main):003:0> Notify.test_email('destination_email@address.com', 'Message Subject', 'Message Body').deliver_now

```

### 三、Gitlab 备份及恢复

`backup`

```

sudo gitlab-rake gitlab:backup:create

可选参数：SKIP=db,upload 或者SKIP=db或者SKIP=upload

```

`resotre`

**a. 停止服务**

```

sudo cp 1393513186_gitlab_backup.tar /var/opt/gitlab/backups/

sudo gitlab-ctl stop unicorn

sudo gitlab-ctl stop sidekiq

```

**b.查看其状态**

```

sudo gitlab-ctl status

```

**c.还原**

```

$ sudo gitlab-rake gitlab:backup:restore BACKUP=1393513186_2014_02_27

 可选参数

 $ sudo gitlab-rake gitlab:backup:restore SKIP=db,upload BACKUP=1342713186_2014_02_27

```

**d. 重启并检查**

```

sudo gitlab-ctl start 

sudo gitlab-rake gitlab:check SANITIZE=true

```

### 四、禁用Gitlab nginx使用production-nginx

**禁用gitlab nginx**

```

vim /etc/gitlab/gitlab.rb

nginx['enable'] = false

```

**重新配置gitlab**

```

gitlab-ctl reconfigure

```

**使用production配置** 
`建议：将/var/opt/gitlab/nginx/conf下面的配置文件直接复制过来，稍作修改即可`
参考配置：http://docs.gitlab.com/omnibus/settings/nginx.html

```
#注意：拷贝前备份
cp /var/opt/gitlab/ngin/conf/* /usr/local/nginx/conf/
```

### 五、禁用Gitlab postgresql使用production-pgsql

**1.禁用Gitlab postgresql**

```

$ vim /etc/gitlab/gitlab.rb

postgresql['enable'] = false

```

**2.导入Gitlab 备份sql到production postgresql**

连接数据库并创建(注意，在最后创建数据库的时候，如果出现报错，请使用gitlab用户登录并创建)

```

$ psql -U PSQL_USER -h PSQL_HOST -p 5432 -d template1

template1 > CREATE USER gitlab CREATEDB WITH PASSWORD 'PASSWORD';

template1 > CREATE EXTENSION IF NOT EXISTS pg_trgm;

template1 > CREATE DATABASE gitlabhq_production OWNER gitlab

# 导入备份文件

psql -U PSQL_USER -h PSQL_HOST -p 5432 -d DATABASE < SQL_FILE

```

**3.修改gitlab配置，添加production**

```
$ vim /etc/gitlab/gitlab.rb
gitlab_rails['db_adapter'] = "postgresql"

gitlab_rails['db_encoding'] = 'utf8'

gitlab_rails['db_host'] = 'HOST'

gitlab_rails['db_port'] = '5432'

gitlab_rails['db_username'] = 'User_Name'

gitlab_rails['db_password'] = 'Password'

```

**4.重新配置Gitlab**

```

gitlab-ct reconfigure

```

### 六、禁用Gitlab Redis使用production Redis

**配置gitlab**

```
$ vim /etc/gitlab/gitlab.rb

redis['enable'] = false

gitlab_rails['redis_host'] = 'REDIS_HOST'

gitlab_rails['redis_port'] = 6379
```

**备份本地redis**

```
redis-cli -h localhost -p 6379

redis 127.0.0.1:6379 > SAVE

OK

该命令将在 redis 安装目录中创建dump.rdb文件。

```

**恢复数据到production**

只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可,获取 redis 目录可以使用 CONFIG 命令，如下

```
redis 127.0.0.1:6379> CONFIG GET dir

1) "dir"

2) "/usr/local/redis/bin"

以上命令 CONFIG GET dir 输出的 redis 安装目录为 /usr/local/redis/bin。

```

**如果还原的redis在Aws的ElastiCache**

在ElastiCache控制面板配置redis(https://console.amazonaws.cn/elasticache)

##### 1.创建redis集群

![gitlab2.png](http://obbogqhb1.bkt.clouddn.com/gitlab2.png)

##### 2.配置基本配置
![gitlab3.png](http://obbogqhb1.bkt.clouddn.com/gitlab3.png)

##### 3.配置高级配置
![gitlab4.png](http://obbogqhb1.bkt.clouddn.com/gitlab4.png)
##### 4. 等待创建完成后重新配置Gitlab

```

gitlab-ctl reconfigure 

```

<iframe height=498 width=510 src='http://player.youku.com/embed/XNTIzNzE2NzQ4' frameborder=0 'allowfullscreen'></iframe>
