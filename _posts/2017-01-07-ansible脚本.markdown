---
layout:     post
title:      "ansible脚本"
subtitle:   "Ansible Script"
date:       2017-01-07
author:     "Antony"
header-img: "img/post-bg-js-module.jpg"
tags:
    - ansible
    - 自动化工具
---
>这是一个可以自动在多台server批量安装及拷贝配置文件启动服务器的ansible脚本，本人学疏才浅，目前还有很多待完善，希望各位多多指教。

```
#!/bin/bash
# Author: Altamob 
# Directory tree
# 	/opt/			
# 	├── ansible.sh	# like this,nginx.conf is varia $FILE
# 	└── nginx.conf

# Host IP One
HOST=172.18.4.62
# Host IP two
HOST1=
# conf file template
FILE=nginx.conf

cat << EOF
+---------------------------------------+
|   this is a ansible script           |
|      from Altamob.......          |
+---------------------------------------
EOF

read -p "pelease input your service name: " SER
read -p "pelease input your hostgroup name: " HOSTGROUP


mkdir /etc/ansible/roles/${SER}/{tasks,templates,vars,handlers} -pv &>/dev/null

cp $FILE /etc/ansible/roles/${SER}/templates/${FILE}.j2 &>/dev/null
cat >> /etc/ansible/hosts << EOF
[$HOSTGROUP]
$HOST
$HOST1
EOF
cat > /etc/ansible/roles/$SER/tasks/main.yml << EOF
- name: install $SER
  yum: name=$SER state=present
- name: copy conf file
  template: src=${FILE}.j2 dest=/etc/${SER}/${FILE}
  notify: restart
  tags: conf
- name: start service
  service: name=nginx state=started 
EOF
#cat > /etc/ansible/roles/$SER/tasks/main.yml << EOF
#- name: install $SER
#  yum: name=$SER state=present
#- name: copy conf file
#  template: src=${FILE}.j2 dest=/etc/${SER}/${FILE}
#  notify: restart
#  tags: conf
#- name: start service
#  service: name=nginx state=started 
#EOF
cat > /etc/ansible/roles/${SER}/handlers/main.yml <<EOF
- name: restart
  service: name=${SER} state=restarted
EOF

cat >> /etc/ansible/ansible.cfg <<EOF
host_key_checking = False
EOF
cat >> /opt/${SER}.yaml <<EOF
- hosts: ${HOSTGROUP}
  remote_user: root
  roles:
  - ${SER}
EOF

cd /opt/;ansible-playbook ${SER}.yaml
```

>作者：`Antony` WX&QQ：`1257465991`
Q/A：如有问题请慷慨提出


