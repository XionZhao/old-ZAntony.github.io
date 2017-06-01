---
layout:     post
title:      "FastDFS unti for systemd"
subtitle:   "FastDFS分布式存储"
date:       2017-01-07
author:     "Antony"
header-img: "img/post-bg-js-module.jpg"
tags:
    - FastDFS
    - 分布式存储
---
### FastDFS tracker的unit脚本
```
# Systemd unit file for default fastdfs_tracker
# 

[Unit]
Description=FastDFS tracker script
After=syslog.target network.target

[Service]
Type=notify
ExecStart=/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf


[Install]
WantedBy=multi-user.target
```
### FastDFS storage的unit脚本
```
# Systemd unit file for default fastdfs storage
# 

[Unit]
Description=FastDFS storage script
After=syslog.target network.target

[Service]
Type=notify
ExecStart=/usr/bin/fdfs_storaged /etc/fdfs/storage.conf


[Install]
WantedBy=multi-user.target
```

### 使用方法
复制到`/usr/lib/systemd/system/`下，建议命名为: `fdfs_storaged.service`和`fdfs_tracker.service`
