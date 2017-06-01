---
layout:     post
title:      "Linux三剑客之grep"
subtitle:   "Grep 常见用法及基础介绍"
date:       2017-01-07
author:     "Antony"
header-img: "img/post-bg-js-module.jpg"
tags:
    - grep
    - 正则
---
### 一、Linux文本处理三剑客
>Linux上有三种常用的文本处理工具，分别为：grep（egrep、fgrep）、sed、awk。今天主要给大家介绍一下三剑客中的第一剑：grep伐木累。
### 二、grep是什么？
>`grep` 全称（Globally search a Regular Expression and Print）是一个文本搜索工具，基于“pattern”（这里指的是过滤模式，多指正则表达式）对给定的文本进行搜索。

>grep家族：
		1.grep：支持使用基本正则表达式；
        2.egrep：支持使用扩展正则表达式；
		3.fgrep：不支持使用正则表达式；
       # 虽然正则表达式有强大的引擎，但是在单独过滤字符上面 fgrep要起到很大作用，这个作用在日志文件小的时候可能体现不出来，但是当文件达到几亿行就能体现出fgrep的搜索效率。（对大型web网站一天的日志量到几亿行是很轻松的，甚至更多）

### 三、grep与egrep的主要参数
在介绍正则表达式前，先介绍一下grep的常用参数，这里所有涉及正则表达式meta字符都会在后续中展开！
常用选项：
--color=auto：对匹配到的文本着色后高亮显示；（默认centos7对grep、egrep、fgrep已经设置参数，此处不做过多例子）
####  alias 
```
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```
#### -i：忽略字符大小写；
```
# cat test
Hello World
How are you?
Mageedu is very good!
# grep "^H" test
Hello World
How are you?
# grep "^h" test
# grep -i "^h" test
Hello World
How are you?
```
#### -o：仅显示匹配 到的文本自身；
```
# grep "good" test
Mageedu is very good!
[root@localhost ~]# grep -o "good" test
good
-v，反向匹配；
# cat test
Hello World
How are you?
Mageedu is very good!
[root@localhost ~]# grep  "M.*[[:punct:]]$" test
Mageedu is very good!
[root@localhost ~]# grep -v "M.*[[:punct:]]$" test
Hello World
How are you?
```
#### -q, 静默模式，不输出任何信息；（省得把输出放到/dev/null，节省系统资源）
此处只是为了得到一个返回状态
```
# grep -q "good" test
# echo $?
0
# grep -q "goodddddddd" test
# echo $?
1
```
#### -P, 支持使用pcre正则表达式（per语言）；
#### -e PATTERN可以同时匹配多个模式
```
# cat test
Hello World
How are you?
Mageedu is very good!
[root@localhost ~]# grep -e "good" -e "you" test
How are you?
Mageedu is very good!
```
#### -f FILEFILE为每行包含了一个pattern的文本文件，即grep script；（读取文件中的每一行pattern来对指定FILE进行过滤）
-f与-e类似，都是匹配多个模式，只是-f选项读取的是文本文件中的模式，逐行匹配。
#### -A NUM 显示匹配出的后#行
#### -B NUM 显示匹配出的前#行
#### -C NUM显示匹配出的前后#行
```
# cat test
Hello World
How are you?
Mageedu is very good!
# grep -A 1 "you" test
How are you?
Mageedu is very good!
# grep -B 1 "you" test
Hello World
How are you?
# grep -C 1 "you" test
Hello World
How are you?
Mageedu is very good!
```
#### 其他参数
下面参数针对grep命令而言：
-E：支持扩展的正则表达式相当与egrep；
-F, --fixed-strings：支持使用固定字符串，不支持正则表达式，相当于fgrep；

### 四、grep使用及正则表达式
#### 1.正则表达式的组成
a、一般字符：没有特殊意义的字符
b、特殊字符（meta字符）：元字符，有在正则表达式中有特殊意义
#### 2.下面讲下正则表达式中常见的meta字符(元字符)
##### a、基本正则表达式meta字符
##### （1）字符匹配
###### ".":匹配任意单个字符；
```
# grep r..t /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
```
###### [ ]:匹配方位内的任意单个数字；
```
# fdisk -l |grep "[sh]d[a-z]\>"
Disk /dev/sda: 128.8 GB, 128849018880 bytes, 251658240 sectors
```
###### [^]匹配范围外的任意单个数字：托字符指取反之意,下面会讲到
```
# cat test
Hello World
how are you?
Mageedu is very good!
```
###### [[:upper:]]：所有大写字母
````
# grep "^[^[:upper:]]" test
how are you?
```
###### 其他表示数字的元字符
[[:upper:]]：所有大写字母；
[[:lower:]]：所有小写字母；
[[:digit:]]：所有的数字；
[[:alpha:]]：所有字母；
[[:alnum:]]：所有字母和数字；
[[:space:]]：空白字符；
[[:punct:]]：标点符号；
```
# cat test
Hello World
how are you?
Mageedu is very good!
# grep "[[:upper:]][[:lower:]].*[[:space:]]" test
Hello World
Mageedu is very good!
```
###### （2）匹配次数
主要用于限制字符要出现的次数，一般放在出现的次数的字符后面。
######   `*`：匹配前面的字符任意次（0,1或多次）；
此处*表示前面的字符可以出现也可以不出现，所以将文件中所有的行都匹配出来了
```
# grep "a*" test1
ab
aaaabcd
bcd
cd
axxxxxxxd
```
###### `.*`：任意长度的任意字符（.为匹配单个字符，*为任意次）
此处.表示匹配任意单个字符，所以在过滤时，文件中的a必须出现至少一次
```
# grep "a.*" test1
ab
aaaabcd
axxxxxxxd
cccabbbd
```
######  `\+`：匹配前面的字符至少1次；
```
# grep "a\+" test1
ab
aaaabcd
axxxxxxxd
cccabbbd
```
######  \?：匹配前面的0次或1次，即前面的字符可有可无；
```
# grep "a\?" test1
ab
aaaabcd
bcd
cd
axxxxxxxd
cccabbbd
```
###### \{\m}：固定匹配前面的字符出现m次，m为非负数；
```
# grep "a\{2\}" test1
aaaabcd
aaxxxxxxxd
cccaabbbd
```
###### \{m,n\}:至少出现m次，至多出现n次；
```
# grep "a\{1,3\}" test1
ab
aaaabcd
aaxxxxxxxd
cccaabbbd
    \{0,n\}:至多n次；
    \{m,\}：至少m次；
```
##### （3）位置锚定
限制使用模式搜索文本，限制模式所匹配到的文本只能出现于目标文本的哪个位置；

`^`：行首锚定；用于模式的最左侧，^PATTERN
`$`：行尾锚定；用于模式的最右侧，PATTERN$
```
# ps -ef|grep "^root.*bash$"
root       1143   1141  0 09:20 pts/0    00:00:00 -bash
^PATTERN$：要让PATTERN完全匹配一整行；
# grep "^hello" /tmp/fstab
hello
hello world
# grep "^hello$" /tmp/fstab
hello
```
###### ^$：空行；
```
常用语在文档中过滤多余的空行，-v指取反之意
# cat test
Hello World

how are you?

Mageedu is very good!

# grep -v "^$" test
Hello World
how are you?
Mageedu is very good!
```
###### ^[[:space:]]*$：用来在^$匹配不到的时候使用；
此处指有的时候空行中包含空白字符，比如空格、制表符等，用^$就过滤不出来了，此处可以使用此命令过滤
```
# grep -v "^$" test
Hello World
  
      
how are you?
       
       
Mageedu is very good!
# grep -v "^[[:space:]]*$" test
Hello World
how are you?
Mageedu is very good!
```
###### \<或\b：词首锚定，用于单词模式的左侧，格式为\<PATTERN, \bPATTERN
```
# fdisk -l|grep "/dev/[sh]d[a-z]"
Disk /dev/sda: 128.8 GB, 128849018880 bytes, 251658240 sectors
/dev/sda1   *        2048     1026047      512000   83  Linux
/dev/sda2         1026048    84912127    41943040   83  Linux
/dev/sda3        84912128   126855167    20971520   83  Linux
/dev/sda4       126855168   251658239    62401536    5  Extended
/dev/sda5       126859264   131055615     2098176   82  Linux swap / Solaris
# fdisk -l|grep "/dev/[sh]d[a-z]\>"
Disk /dev/sda: 128.8 GB, 128849018880 bytes, 251658240 sectors
```
###### \>或\b：词尾锚定，用于单词模式的右侧，格式为PATTERN\>, PATTERN\b
```
# grep "\<root" /etc/passwd
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
```
###### \<PATTERN\>：单词锚定
```
单词：由非特殊字符组成的连续字符（字符串）都称为单词；
# cat /tmp/ifconfig.test 
/usr/sbin/ifconfig
/usr/sbin/ifconfig2
/usr/sbin/ifconfig3
# grep "ifconfig" /tmp/ifconfig.test 
/usr/sbin/ifconfig
/usr/sbin/ifconfig2
/usr/sbin/ifconfig3
# grep "\<ifconfig\>" /tmp/ifconfig.test 
/usr/sbin/ifconfig
```
##### （4）分组与引用
\(PATTERN\):将次PATTERN匹配到的字符当做一个整体处理；分组括号中的模式匹配到的字符会被正则表达式引擎自动纪录于变量中，这些变量分别是\1，\2，\3
    \1：第一组括号中的pattern匹配到的字符串；
    \2：第二组括号中的pattern匹配到的字符串；
```
# grep "^\([[:alnum:]]\+\)\>.*\1$" /etc/passwd
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
nologin:x:5010:5010::/home/nologin:/sbin/nologin
```
##### b、扩展正则表达式meta字符
注意：与基本正则表达式不同，扩展表达式在一些参数上可以不使用转意符（\），但是在使用上没有什么区别，这里我举几个例子，其他基本与基本正则表达式使用相同！      

##### 此处表示过滤1-255的所有整数
![enter image description here](http://zantony.top/uploads/grep.png)
##### 此处表示取出绝对路径中的基名
```
# echo `which pwd`|egrep "[^/]+/?$" 
/usr/bin/pwd
# echo `which pwd`|egrep "[^/]+/?$" -o
pwd
```
##### 此处表示查看root、mageedu、centos用户的详细信息
```
# egrep "^(root|mageedu|centos)\>" /etc/passwd
root:x:0:0:root:/root:/bin/bash
mageedu:x:1000:1000:mageedu:/home/mageedu:/bin/bash
centos:x:5008:5008::/home/centos:/bin/bash
```
#### 总结
（1）字符匹配
.:匹配任意单个字符；
`[ ]`:匹配方位内的任意单个数字；
`[^]`匹配范围外的任意单个数字；
`[[:upper:]]`：所有大写字母；
`[[:lower:]]`：所有小写字母；
`[[:digit:]]`：所有的数字；
`[[:alpha:]]`：所有字母；
`[[:alnum:]]`：所有字母和数字；
`[[:space:]]`：空白字符；
`[[:punct:]]`：标点符号；
（2）匹配次数
主要用于限制字符要出现的次数，一般放在出现的次数的字符后面。
`*`：匹配前面的字符任意次（0,1或多次）；
`.*`：任意长度的任意字符（.为匹配单个字符，*为任意次）
`+`：匹配前面的字符至少1次；
`?`：匹配前面的0次或1次，即前面的字符可有可无；
`{m}`：固定匹配前面的字符出现m次，m为非负数；
`{m,n}`:至少出现m次，至多出现n次；
`{0,n}`:至多n次；
`{m,}`：至少m次；
（3）位置锚定
扩展正则表达式中：单词锚定符（\<,\>）不能去除转意符(\)
`^`：行首锚定；用于模式的最左侧，^PATTERN
`$`：行尾锚定；用于模式的最右侧，PATTERN$
`^PATTERN$`：要让PATTERN完全匹配一整行；
`^$`：空行；
`^[[:space:]]*$`: 用来在^$匹配不到的时候使用；
`\<或\b`：词首锚定，用于单词模式的左侧，格式为\<PATTERN, \bPATTERN
`\>或\b`：词尾锚定，用于单词模式的右侧，格式为PATTERN\>, PATTERN\b
`\<PATTERN\>`：单词锚定
单词：由非特殊字符组成的连续字符（字符串）都称为单词；
（4）分组与引用
(PATTERN):将次PATTERN匹配到的字符当做一个整体处理；分组括号中的模式匹配到的字符会被正则表达式引擎自动纪录于变量中，这些变量分别是\1，\2，\3
`\1`：第一组括号中的pattern匹配到的字符串；
` \2`：第二组括号中的pattern匹配到的字符串
这里的\1 \2不能省略转意符；
（5）或者
   ` a|b`：a或者b
    `C|cat`：表示C或cat
   `(C|c)at`：表示Cat或cat
```
# fdisk -l|grep -E "/dev/(s|h)d[a-z]\>"
Disk /dev/sda: 128.8 GB, 128849018880 bytes, 251658240 sectors
```
>作者：Antony WX&QQ：1257465991
Q/A：如有问题请慷慨提出
