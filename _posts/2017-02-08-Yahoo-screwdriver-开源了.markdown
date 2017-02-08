---
layout:     post
title:      "Gitlab-Omnibus"
subtitle:   "Yahoo-screwdriver开源了!"
date:       2017-02-08
author:     "Antony"
header-img: "img/yahoo.jpg"
tags:
    - CI(持续集成)
    - CD(持续交付)
---
### screwdriver 介绍

**screwdriver 是yahoo开源的一款发布工具，用于大规模持续交付到生产的动态基础架构.

这应该是目前最完整拥有CI(持续集成Continuous integration)和CD(持续交付Continuous delivery)的专案.**

- **官网**: http://screwdriver.cd/

- **Doc**: http://docs.screwdriver.cd/user-guide/quickstart/

- **Github**: https://github.com/screwdriver-cd

- **Docker**: https://hub.docker.com/u/screwdrivercd/

#### 特色

- 1.使部署通道更容易 (Making deployment pipelines easy)


- 2.优化主干开发(Optimizing for trunk development)

- 3.回滚更加容易(Making rolling back easy)

详情请阅: https://yahooeng.tumblr.com/post/155765242061/open-sourcing-screwdriver-yahoos-continuous

### Screwdrive 架构和开发流程

详情请查看：http://docs.screwdriver.cd/cluster-management/

#### 组件:

- 1.REST API(与流水线协同工作的接口)

- 2.Web UI(用于流水线API的可视化接口)

- 3.Launcher（启动器）: 设置环境并执行shell命令的工具

- 4.Execution Engine（执行引擎）：可插拔的构建执行器，支持在容器（Jenkins、Kubernetes、Mesos、Docker Swarm）内执行命令。

- 5.Datastore（数据存储）：可插拔的NoSQL存储，用于维护流水线配置数据（DynamoDB、MongoDB、CouchDB、Postgres）。执行引擎和数据存储都使用了可插拔的架构，使得用户可按自身意向选用引擎。
![Yahho png](http://obbogqhb1.bkt.clouddn.com/yahoo.png)
>作者：Antony WX&QQ：1257465991
Q/A：如有问题请慷慨提出
