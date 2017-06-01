---
layout:     post
title:      "aws自定义AMI-UUID相同处理方法"
subtitle:   "实例都是从同一个AMI 上创建"
date:       2017-01-12
author:     "Antony"
header-img: "img/aws-interface.jpg"
tags:
    - aws
---
> Note: 在制作Centos 7 AMI并进行使用时，我们发现：如果将两个相同AMI挂载到一个操作系统的时候，会出现因为UUID相同，导致不能挂载的问题(至于为什么将两个相同的AMI挂载到一个操作系统：当误删除用户的密钥文件或者不能登录系统时，我们不得不将其挂载到其他操作系统)

#### 问题重现

```

$ sudo fdisk -l

Disk /dev/xvda: 53.7 GB, 53687091200 bytes, 104857600 sectors

Units = sectors of 1 * 512 = 512 bytes

Sector size (logical/physical): 512 bytes / 512 bytes

I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk label type: dos

Disk identifier: 0x0001783d



    Device Boot      Start         End      Blocks   Id  System

/dev/xvda1   *        2048   104856254    52427103+  83  Linux



Disk /dev/mapper/docker-202:1-84149041-pool: 107.4 GB, 107374182400 bytes, 209715200 sectors

Units = sectors of 1 * 512 = 512 bytes

Sector size (logical/physical): 512 bytes / 512 bytes

I/O size (minimum/optimal): 65536 bytes / 65536 bytes





Disk /dev/xvdf: 53.7 GB, 53687091200 bytes, 104857600 sectors

Units = sectors of 1 * 512 = 512 bytes

Sector size (logical/physical): 512 bytes / 512 bytes

I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk label type: dos

Disk identifier: 0x0001783d



    Device Boot      Start         End      Blocks   Id  System

/dev/xvdf1   *        2048   104856254    52427103+  83  Linux



# 挂载新硬盘

$ sudo mount /dev/xvdf1 /mnt/

# 此时出现问题，通过dmesg查看到是因为uuid已经存在。

mount: wrong fs type, bad option, bad superblock on /dev/xvdf1,

       missing codepage or helper program, or other error



       In some cases useful info is found in syslog - try

       dmesg | tail or so.

[meiqia-sa@jenkins-app0 ~]$ dmesg |tail

[21609.549695]  xvdf: xvdf1

[21737.056975] XFS (xvdf1): Filesystem has duplicate UUID test-uuid-c888-4f55-9d74-e3495a50ba03 - can't mount

```

#### 解决方法

```

$ sudo mount -o nouuid /dev/xvdf1 /mnt/

$ df -hT

Filesystem     Type      Size  Used Avail Use% Mounted on

/dev/xvda1     xfs        50G  8.2G   42G  17% /

/dev/xvdf1     xfs        50G  5.9G   45G  12% /mnt

```
>作者：`Antony` WX&QQ：`1257465991`
Q/A：如有问题请慷慨提出


