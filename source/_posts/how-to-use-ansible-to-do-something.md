---
title: 使用Ansible批量操作服务器
copyright: true
date: 2018-07-16 22:23:51
tags: Ansible
categories: Linux
---

# 使用Ansible批量操作服务器

Anbible 是干嘛的？对于一个非专业运维人士（我）来说，它就是我批量操作服务器的一个神器。试想一个场景：公司内部DNS还未搭建好，业务系统使用了域名做请求，这个时候需要你将域名制定到某台机器上，你这个时候就只能修改hosts文件了。嗯，如果是一两台服务器就算了，大不了手动ssh上去改一改，但是如果是10台了？10台不够，100台了？这个时候怎么办？你可能说我召集了一帮兄弟，大家一人改几个。OK，好不容易你改完了，这个时候业务跟你说，嗯，那台服务器挂了，不稳定，暂时换到另一台服务器上另一个IP地址。兄弟，听说醉经淘宝刀打折，买一把吧。哈哈

但是如果这个时候我们使用ansible，这个时候你就可以早点干完，早点回家陪老婆孩子了。接下来我们看看怎么使用ansible，请注意，我只是说怎么使用，是的，怎么使用，没有任何理论，不会讲解任何深的东西，只是用而已。
<!--more-->
1：批量操作服务器。

ansible的一个简单目录结构：
![2018-07-16-22-34-50](/images/qiniu/2018-07-16-22-34-50.png)

`install.yml`文件内容：

![2018-07-16-22-36-22](/images/qiniu/2018-07-16-22-36-22.png)

<!-->
```
---
- hosts: all 
  remote_user: rhlog
  gather_facts: no
  roles:
    - role: oam
      become: yes
      become_method: su
      become_user: root
```
<-->

`hosts` 文件内容：

```
10.0.80.34 ansible_become=true ansible_become_method=su ansible_user=ssh_user ansible_ssh_pass="password" ansible_become_pass="ax=n@#*!EM" ansible_become_user=root
10.07.80.37 ansible_become=true ansible_become_method=su ansible_user=ssh_user ansible_ssh_pass="W)PIukAa" ansible_become_pass="r.*o)Hg!z" ansible_become_user=root
```

这两个是ansible的根目录里面比较重要的文件。install.yml里面remote_user是指登录到服务器的用户，通常大家不会让root用户远程登录的。roles是指在roles文件里面有哪些角色，你要用使用哪个角色。hosts文件一般保存的账号密码。按照这个模式登录就好了。之后我会提供这个安装包，大家可以自己照着改。哈哈。

oam角色里文件内容为：
![2018-07-16-22-41-50](/images/qiniu/2018-07-16-22-41-50.png)
主要看tasks目录里面：

```
---
# tasks file for oam
- name: upload file
  copy: src=/app/installation/resource/moving/test/roles/oam/files/data.sh dest=/home/data.sh mode=755
- name: excute 
  shell:  bash /home/data.sh
- name: find file 
  find:
    paths: /home/
    patterns: "*.log"
    recurse: no
  register: file_2_fetch
- name: fetch
  fetch:
    src: "{{ item.path }}"
    dest: /home/diskdata/
    flat: yes
  with_items: "{{ file_2_fetch.files }}"
```
<!-->
```
# !/bin/bash
name=$(hostname -i)
file=${name%% *}
df -h >/home/rhlog/$file.log
```
<-->
在ansible根目录我怎么使用了？`ansible-playbook -i hosts install.yml` 我的这个程序就是将files目录下的data.sh上传到hosts里面的所有主机中，然后执行data.sh，将`df -h`的内容输出到`ip.log`文件中，然后将文件下载到本地`/home/diskdata`中。这样就相当于批量操作了所有的机器。


另外一个使用方式，不使用上面那种方式，而是直接使用命令行的方式：

1： 判断所有机器是否可以访问某个地址

```
ansible all -i hosts -m get_url -a "url=http://192.168.11.32:8088/srv/releaseService?wsdl dest=./"
```

2: 判断所有机器是否可以ping通某个地址

```
ansible all -i hosts -m shell -a "ping -c 3 10.7.18.3"
```

3: 查找所有机器上某个应用是否启动

```
ansible all -i hosts -m shell -a "ps -ef | grep haproxy | grep -v grep"
```

4: 判断所有机器能否连通某个端口

```
ansible all -i hosts -m shell -a "telnet 192.167.180.3 25"
```

5: 批量修改某个文件

```
ansible all -i ./hosts -s -m shell -a "echo \"127.0.0.1 test.local.com\" >> /etc/hosts"
```

ps:

`-s`,是指使用root账户