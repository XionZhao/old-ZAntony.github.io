---
layout:     post
title:      "制作并安装FastDFS的RPM程序包"
subtitle:   "FastDFS rpm"
date:       2017-01-07
author:     "Antony"
header-img: "img/post-bg-js-module.jpg"
tags:
    - FastDFS rpm
---
### 一、安装并制作rpm包libfastcommon
`注意`:生成的rpm包全都在`/root/rpmbuild/RPMS/`下面，可以再其他主机直接使用
```
#!/bin/bash
# Author: Antony
# rpm build for fastdfs
# Mail: zhaoxin@altamob.com,go80800@163.com
# Desc: build libfastcommmon # 制作和安装libfastcommon

cat << EOF
+-------------------------------------------+
|   this is a rpmbuild libfastcommon script |
|          from Antony .......              |
+--------------------------------------------
EOF
# 制作源码文件 建议手动安装yum
#yum groupinstall "Development Tools" "Server platform Development" -y 

cd /root/

git clone https://github.com/happyfish100/libfastcommon.git 

if [ $? -ne 0 ];then
exit 1
fi

libversion=$(grep -i "^version" libfastcommon/libfastcommon.spec |awk -F':' '{print $2}'|awk '{print $1}')

mv libfastcommon/ libfastcommon-$libversion

tar zcf libfastcommon-${libversion}.tar.gz libfastcommon-$libversion 
# 制作rpm包

cd /root
mkdir rpmbuild/{SOURCES,SPECS} -pv
cp libfastcommon-${libversion}.tar.gz rpmbuild/SOURCES/
cp libfastcommon-${libversion}/libfastcommon.spec rpmbuild/SPECS/

rpmbuild -ba /root/rpmbuild/SPECS/libfastcommon.spec &>/dev/null

yum install /root/rpmbuild/RPMS/x86_64/libfastcommon-1.0.29-1.el7.centos.x86_64.rpm /root/rpmbuild/RPMS/x86_64/libfastcommon-devel-1.0.29-1.el7.centos.x86_64.rpm -y &>/dev/null

sleep 2
if rpm -q libfastcommon &>/dev/null;then

cat << EOF
+---------------------------------------+
|                                       |
|        libfastcommon  OK              |
|                                       |
+---------------------------------------
EOF

else 
	echo "install filed!"
fi
```
### 二、制作并安装FastDFS
`注意`:生成的rpm包全都在`/root/rpmbuild/RPMS/`下面，可以再其他主机直接使用
第二步骤，如果报错，请再执行一遍。
```
#!/bin/bash
# Author: Antony
# rpm build for fastdfs
# Mail: zhaoxin@altamob.com,go80800@163.com
# Desc: build fastdfs # 作fastdfs rpm
cat << EOF
+-------------------------------------------+
|   this is a rpmbuild FastDFS   script     |
|          from Antony .......              |
+--------------------------------------------
EOF
if rpm -q libfastcommon;then
	
cd /root/

git clone https://github.com/happyfish100/fastdfs.git

if [ $? -ne 0 ];then
exit 1
fi

sleep 3

fdfsv=$(grep -E "FDFSVersion[[:space:]]+" fastdfs/fastdfs.spec|awk '{print $3}')

mv fastdfs fastdfs-${fdfsv}

cd /root/

tar zcf fastdfs-${fdfsv}.tar.gz fastdfs-${fdfsv}/

mkdir rpmbuild/{SOURCES,SPECS} -pv
cp fastdfs-${fdfsv}.tar.gz rpmbuild/SOURCES/
cp fastdfs-${fdfsv}/fastdfs.spec rpmbuild/SPECS/
rpmbuild -ba /root/rpmbuild/SPECS/fastdfs.spec 

yum install /root/rpmbuild/RPMS/x86_64/fastdfs-5.0.8-1.el7.centos.x86_64.rpm /root/rpmbuild/RPMS/x86_64/fastdfs-server-5.0.8-1.el7.centos.x86_64.rpm /root/rpmbuild/RPMS/x86_64/fastdfs-tool-5.0.8-1.el7.centos.x86_64.rpm /root/rpmbuild/RPMS/x86_64/libfdfsclient-5.0.8-1.el7.centos.x86_64.rpm /root/rpmbuild/RPMS/x86_64/libfdfsclient-devel-5.0.8-1.el7.centos.x86_64.rpm -y &>/dev/null

if [ $? -eq 0 ];then
cat << EOF
+---------------------------------------+
|                                       |
|        Everything  OK                 |
|                                       |
+----------------------------------------
EOF
fi

else
        echo "libcommon is not install"
fi

```

>作者：Antony WX&QQ：1257465991
Q/A：如有问题请慷慨提出
