---
layout:     post
title:      "基于amazon-s3创建yum仓库"
subtitle:   "yum storage to amzon s3"
date:       2017-03-17
author:     "Antony"
header-img: "img/yum-s3.jpg"
tags:
    - aws-s3
    - yum
---
## AWS 配置

### 1.创建s3 仓库

```

aws s3 mb s3://yum-repo-s3

make_bucket: yum-repo-s3

```



### 2.创建s3用户

```

$~ aws iam create-user --user-name yum-repo-user

```

### 3.赋予s3用户权限

```

$ aws iam put-user-policy --user-name yum-repo-user --policy-name yum-repo-s3-Bucket-Access --policy-document file://demo-s3-rpm-repo_policy.json

$ cat demo-s3-rpm-repo_policy.json

{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Sid": "Stmt1489722431765",

      "Action": "s3:*",

      "Effect": "Allow",

      "Resource": "arn:aws-cn:s3:::yum-repo-s3"

    }

  ]

}

```

### 4.查看其权限

```


$ aws iam list-user-policies --user-name yum-repo-user

{

    "PolicyNames": [

        "yum-repo-s3-Bucket-Access"

    ]

}



```

### 5.创建access key

```

$ aws iam create-access-key --user-name yum-repo-user

{

    "AccessKey": {

        "UserName": "yum-repo-user",

        "Status": "Active",

        "AccessKeyId": "KEYID",

        "SecretAccessKey": "SECRET_KEY",

        "CreateDate": "2017-03-17T04:16:55.969Z"

    }

}

```

## 仓库配置

### 1.安装需要的程序包

```

$ sudo yum groupinstall "Development Tools" -y

$ sudo yum install s3cmd --enablerepo=epel -y

$ sudo yum install createrepo pinentry -y

```

### 2.添加用户(此处也可以使用root)

```

$ sudo adduser builder

$ sudo echo -e "builder\tALL=(ALL) NOPASSWD:ALL">/etc/sudoers.d/builder

```

### 3.配置用户access key

```

$ aws configure

AWS Access Key ID [None]: KEY

AWS Secret Access Key [None]: SECRET_KEY

Default region name [None]: cn-north-1

Default output format [None]: json

```

### 4.配置一个专用的Public/Private GPG密钥对

**生成一个新的GPG密钥对**

这里要等一段时间，他需要从随机数库里面去数字，在此期间可以敲击键盘或者点击鼠标来生成随机数(注意：不要ctrl+c)

```

gpg --gen-key

gpg (GnuPG) 2.0.22; Copyright (C) 2013 Free Software Foundation, Inc.

This is free software: you are free to change and redistribute it.

There is NO WARRANTY, to the extent permitted by law.



Please select what kind of key you want:

   (1) RSA and RSA (default)

   (2) DSA and Elgamal

   (3) DSA (sign only)

   (4) RSA (sign only)

Your selection?

RSA keys may be between 1024 and 4096 bits long.

What keysize do you want? (2048)

Requested keysize is 2048 bits

Please specify how long the key should be valid.

         0 = key does not expire

      <n>  = key expires in n days

      <n>w = key expires in n weeks

      <n>m = key expires in n months

      <n>y = key expires in n years

Key is valid for? (0)

Key does not expire at all

Is this correct? (y/N) y



GnuPG needs to construct a user ID to identify your key.



Real name: repo-for-me

Email address: repo@repo.com

Comment: my repo

You selected this USER-ID:

    "repo-for-me (my repo) <repo@repo.com>"



Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o

You need a Passphrase to protect your secret key.



We need to generate a lot of random bytes. It is a good idea to perform

some other action (type on the keyboard, move the mouse, utilize the

disks) during the prime generation; this gives the random number

generator a better chance to gain enough entropy.

```

**查看生成的GPG密钥对**

```

# gpg --list-keys

/root/.gnupg/pubring.gpg

------------------------

pub   2048R/AE355D 2017-03-17

uid                 repo-for-me (my repo) <repo@repo.com>

sub   2048R/45DEQAD 2017-03-17

```

**导出public key**

```

gpg --output ~/RPM-GPG-KEY-my-repo --armor --export AE355D

```

**导出private key**

```

gpg --output ~/RPM-GPG-KEY-my-repo.private --export-secret-key -armor AE355D

```



**rpm 导入key**

```

rpm --import ~/RPM-GPG-KEY-my-repo

```

**确认被正确安装**

```

rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n'

gpg-pubkey-f4a80eb5-53a7ff4b --> gpg(CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>)

gpg-pubkey-621e9f35-58adea78 --> gpg(Docker Release (CE rpm) <docker@docker.com>)

gpg-pubkey-4a7cz49d-533129b --> gpg(repo-for-me (my repo) <repo@repo.com>)

```

### 5.构建RPM packages

**创建rpmbuild目录**

```

mkdir -pv ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}

```

**创建一个repo文件**

```

$ cat ~/repo-for-me.repo

[repo-for-me]

name=name=Extra Packages from Celingest Demo S3 RPM Repository - $basearch

baseurl=https://s3.cn-north-1.amazonaws.com.cn/repo-for-me/yum-repo/$basearch/

enabled=1

gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-my-repo

gpgcheck=1

```

**配置.spec文件**

```

$ cat repo-for-me.spec

# S3 RPM Repository configuration files and GPG key



%define name    repo-for-me

%define version   1

%define release   0.1



%define buildroot %{_topdir}/%{name}-%{version}-root



BuildArch:  x86_64

BuildRoot:  %{buildroot}

Summary:    S3 RPM Repository

License:    GPLv3

Name:       %{name}

Version:    %{version}

Release:    %{release}

Group:      Development/Tools



%description

Package containing antony amzon S3 RPM Repository configuration files and GPG key.



%prep

exit 0



%build



%install

rm -rf $RPM_BUILD_ROOT

mkdir -p $RPM_BUILD_ROOT%{_sysconfdir}/yum.repos.d/

mkdir -p $RPM_BUILD_ROOT%{_sysconfdir}/pki/rpm-gpg/

cp -p ~/repo-for-me.repo $RPM_BUILD_ROOT%{_sysconfdir}/yum.repos.d/

cp -p ~/RPM-GPG-KEY-my-repo $RPM_BUILD_ROOT%{_sysconfdir}/pki/rpm-gpg/



%clean

rm -rf $RPM_BUILD_ROOT



%files

%defattr(-,root,root,-)

/etc/yum.repos.d/repo-for-me.repo

/etc/pki/rpm-gpg/RPM-GPG-KEY-my-repo





%changelog

* Fri Mar 17 2017 Builder <my@my.repo.com> 1.0.1

- First release.

```

**配置rpm macros**

```

cat .rpmmacros

%_signature gpg

%_gpg_path /root/.gnupg

%_gpg_name repo-for-me (my repo) <repo@repo.com> # 此处的名字通过# gpg --list-keys 查看uid

%_gpgbin /usr/bin/gpg

%packager meiqia <repo@repo.com>

%_topdir /root/rpmbuild

```

**制作rpm**

```

rpmbuild -v -ba --sign --clean repo-for-me.spec

...

Enter pass phrase:  #这里的密码是gpg --gen-key生成的密码

Pass phrase is good.



/home/root/rpmbuild/SRPMS/repo-for-me-1-0.1.x86_64.rpm

/home/root/rpmbuild/RPMS/noarch/repo-for-me-1-0.1.x86_64.rpm

```

**检查包的正确性**

```

rpm --checksig /home/builder/rpmbuild/SRPMS/repo-for-me-1-0.1.x86_64.rpm

/root/rpmbuild/RPMS/repo-for-me-1-0.1.x86_64.rpm: rsa sha1 (md5) pgp md5 ok

```

### 更新到s3

```

mkdir -pv ~/yum-repo/{x86_64,noarch} # 注意 yum-repo这个目录必须和repo文件中的一一对应

```

**拷贝rpm包到/yum-repo/x86_64**

```

cp /home/root/rpmbuild/SRPMS/*.rpm ~/yum-repo/x86-64

```

**创建metadata file（最重要的一步）**

```

for a in /root/repo-for-me/x86_64; do createrepo -v --update --deltas $a/ ; done

...

```

**上传到s3**

```

aws s3 sync yum-repo/ s3://repo-for-me/yum-repo/

```

**安装并测试**

```

rpm -ivh https://s3.cn-north-1.amazonaws.com.cn/repo-for-me/yum-repo/x86_64/tools/repo-for-me-1-0.1.x86_64.rpm

```

**本文主要介绍如何管理仓库，如制作rpm包、创建依赖关系、上传rpm到s3等，如果想了解实现方式，请查阅：[repo-for-s3](http://wiki.meiqia.com/pages/viewpage.action?pageId=5701661)**

### 公开s3

![s3-yum](http://obbogqhb1.bkt.clouddn.com/yum-s3.png)
