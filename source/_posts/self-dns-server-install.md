---
title: DNS 服务器搭建
copyright: true
date: 2018-06-09 15:24:01
tags:
categories:
---

# 搭建公司内部DNS服务器

平常有很多时候可能会通过改hosts文件的方式来访问自定义的某个域名，但仅限于单机而已。如果能用dns解析的方式来统一管理那将会很方便。

## 搭建 DNS 服务器

实验基于centos7实践。切换到root用户（以下所有操作都在root下执行）：

1. 安装软件：

我们使用dns软件 bind9 系列安装包。

`yum -y install bind*`

安装后的部分日志:

```
Installed:
  bind.x86_64 32:9.9.4-61.el7               bind-chroot.x86_64 32:9.9.4-61.el7        bind-devel.x86_64 32:9.9.4-61.el7       
  bind-dyndb-ldap.x86_64 0:11.1-4.el7       bind-libs.x86_64 32:9.9.4-61.el7          bind-lite-devel.x86_64 32:9.9.4-61.el7  
  bind-pkcs11.x86_64 32:9.9.4-61.el7        bind-pkcs11-devel.x86_64 32:9.9.4-61.el7  bind-pkcs11-libs.x86_64 32:9.9.4-61.el7 
  bind-pkcs11-utils.x86_64 32:9.9.4-61.el7  bind-sdb.x86_64 32:9.9.4-61.el7           bind-sdb-chroot.x86_64 32:9.9.4-61.el7  
  bind-utils.x86_64 32:9.9.4-61.el7        

Dependency Installed:
  postgresql-libs.x86_64 0:9.2.23-3.el7_4                                                                                      

Updated:
  bind-libs-lite.x86_64 32:9.9.4-61.el7                           bind-license.noarch 32:9.9.4-61.el7       
  ```

<!--more-->

2. 备份主配置文件：

  `cp /etc/named.conf /etc/name.conf.bak.$(date '+%Y-%m-%d')`

3. 修改主配置文件：`vi /etc/named.conf`

  修改两处为`any`：

  ```conf
  options {
	listen-on port 53 { any; }; // 修改此处为any
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	allow-query     { any; }; //修改此处为any
	//forwarders {
		
	//}

	/* 
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable 
	   recursion. 
	 - If your recursive DNS server has a public IP address, you MUST enable access 
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification 
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface 
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation no; //修改这里ping通外网

	/* Path to ISC DLV key */
	bindkeys-file "/etc/named.iscdlv.key";

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
  ```

# 自定义域名配置

现在安装后基本的dns服务器之后，我们就开始自定义个域名来进行解析:

`vi named.rfc1912.zones`

增加一个需要解析的主域名比如：`cococzj.com`;(公网肯定没有这个域名)

增加下面的文件到文件`named.rfc1912.zones`的最后
```
zone "cococzj.com" IN {
        type master;
        file "cococzj.com.zone";
};
```

# 增加域名配置

上面定义了`cococzj.com.zone`的文件，现在增加配置：`vi /var/named/cococzj.com.zone`

增加下面的内容：

```

$TTL 86400
@       IN SOA          ns.cococzj.com. root (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        0 )     ; minimum  

@       IN      NS      ns.cococzj.com.
ns      IN      A       10.62.12.24
www     IN      A       10.62.14.80
ttt     IN      A       10.62.12.3

```

ns 对应的ip地址必须为dns服务器搭建的IP地址，也就是dns安装的机器的ip地址。

`ns      IN      A       10.62.12.24`  ==> `ns      IN      A       your_dns_server_ip`

`www     IN      A       10.62.14.80` 代表将`www.cococzj.com`解析到`10.62.14.80`服务器上。

# 修改zone文件权限

`chown .named /var/named/cococzj.com.zone`

# 检查配置文件

`named-checkconf` 检查主配置文件是否配置正确，没有输出表明是正确的:

`named-checkzone` 检查zone文件配置：

```shell
named-checkzone "cococzj.com" /var/named/cococzj.com.zone

result:
zone cococzj.com/IN: loaded serial 1
OK
```

# 重启服务

可以先ping一下服务域名再重启：
`systemctl restart named.service`


# 测试：

1. 先ping试一下：

```
[root@k8s-dev-yw-1 etc]# ping www.cococzj.com
ping: www.cococzj.com: Name or service not known
```

2. 增加dns服务地址；`vi /etc/resolv.conf`，在文件中新建：

`nameserver 10.62.12.2`

再一次ping：

```shell
[root@k8s-dev-yw-1 etc]# vi /etc/resolv.conf
[root@k8s-dev-yw-1 etc]# ping www.cococzj.com
PING www.cococzj.com (10.62.14.80) 56(84) bytes of data.
64 bytes from 10.62.14.80 (10.62.14.80): icmp_seq=1 ttl=63 time=0.501 ms
64 bytes from 10.62.14.80 (10.62.14.80): icmp_seq=2 ttl=63 time=0.466 ms
64 bytes from 10.62.14.80 (10.62.14.80): icmp_seq=3 ttl=63 time=0.509 ms
^C
--- www.cococzj.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.466/0.492/0.509/0.018 ms
```

# 新增一个域名 

1. `vi named.rfc1912.zones`在文件中新增：

```
zone "your_website.com" IN {
        type master;
        file "your_website.com.zone";
};
```

2. 复制一份之前配置好的配置`cp /var/named/cococzj.com.zone /var/named/your_website.com.zone`

3. 编辑配置文件 `vi /var/named/your_website.com.zone`将`cococzj`改为`your_website`，然后修改相应的ip和

4. 检查配置文件：

```
[root@k8s-dev-yw-1 etc]# named-checkzone "cmrrancher.com" /var/named/chenzhijunrancher.com.zone
zone cmrrancher.com/IN: loaded serial 1
OK
[root@k8s-dev-yw-1 etc]# named-checkconf
[root@k8s-dev-yw-1 etc]# 
```

5. 重载服务
`systemctl reload named.service`

6. 测试：`ping www.chenzhijunrancher.com`


### 附录

```log
[root@qianyi-monitor-3 etc]# yum install -y bind*
Loaded plugins: fastestmirror
http://10.0.0.5/centos/7/cloud/x86_64/openstack-newton/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
To address this issue please refer to the below knowledge base article 

https://access.redhat.com/articles/1320623

If above article doesn't help to resolve this issue please create a bug on https://bugs.centos.org/

http://10.0.0.5/centos/7/docker/x86_64/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
dockerrepo                                                                                                                                          | 2.9 kB  00:00:00     
http://10.0.0.5/centos/7/latest/x86_64/repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
Trying other mirror.
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package bind.x86_64 32:9.9.4-61.el7 will be installed
--> Processing Dependency: libGeoIP.so.1()(64bit) for package: 32:bind-9.9.4-61.el7.x86_64
---> Package bind-chroot.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-devel.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-dyndb-ldap.x86_64 0:11.1-4.el7 will be installed
---> Package bind-libs.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-libs-lite.x86_64 32:9.9.4-29.el7 will be updated
---> Package bind-libs-lite.x86_64 32:9.9.4-61.el7 will be an update
---> Package bind-license.noarch 32:9.9.4-29.el7 will be updated
---> Package bind-license.noarch 32:9.9.4-61.el7 will be an update
---> Package bind-lite-devel.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-pkcs11.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-pkcs11-devel.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-pkcs11-libs.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-pkcs11-utils.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-sdb.x86_64 32:9.9.4-61.el7 will be installed
--> Processing Dependency: libpq.so.5()(64bit) for package: 32:bind-sdb-9.9.4-61.el7.x86_64
---> Package bind-sdb-chroot.x86_64 32:9.9.4-61.el7 will be installed
---> Package bind-utils.x86_64 32:9.9.4-61.el7 will be installed
--> Running transaction check
---> Package GeoIP.x86_64 0:1.5.0-11.el7 will be installed
---> Package postgresql-libs.x86_64 0:9.2.23-3.el7_4 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================================================================================================
 Package                                        Arch                                Version                                        Repository                         Size
===========================================================================================================================================================================
Installing:
 bind                                           x86_64                              32:9.9.4-61.el7                                base                              1.8 M
 bind-chroot                                    x86_64                              32:9.9.4-61.el7                                base                               87 k
 bind-devel                                     x86_64                              32:9.9.4-61.el7                                base                              399 k
 bind-dyndb-ldap                                x86_64                              11.1-4.el7                                     base                              122 k
 bind-libs                                      x86_64                              32:9.9.4-61.el7                                base                              1.0 M
 bind-lite-devel                                x86_64                              32:9.9.4-61.el7                                base                              308 k
 bind-pkcs11                                    x86_64                              32:9.9.4-61.el7                                base                              298 k
 bind-pkcs11-devel                              x86_64                              32:9.9.4-61.el7                                base                              105 k
 bind-pkcs11-libs                               x86_64                              32:9.9.4-61.el7                                base                              1.1 M
 bind-pkcs11-utils                              x86_64                              32:9.9.4-61.el7                                base                              198 k
 bind-sdb                                       x86_64                              32:9.9.4-61.el7                                base                              353 k
 bind-sdb-chroot                                x86_64                              32:9.9.4-61.el7                                base                               87 k
 bind-utils                                     x86_64                              32:9.9.4-61.el7                                base                              204 k
Updating:
 bind-libs-lite                                 x86_64                              32:9.9.4-61.el7                                base                              734 k
 bind-license                                   noarch                              32:9.9.4-61.el7                                base                               85 k
Installing for dependencies:
 GeoIP                                          x86_64                              1.5.0-11.el7                                   base                              1.1 M
 postgresql-libs                                x86_64                              9.2.23-3.el7_4                                 base                              234 k

Transaction Summary
===========================================================================================================================================================================
Install  13 Packages (+2 Dependent packages)
Upgrade   2 Packages

Total download size: 8.1 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/17): bind-9.9.4-61.el7.x86_64.rpm                                                                                                                | 1.8 MB  00:00:00     
(2/17): GeoIP-1.5.0-11.el7.x86_64.rpm                                                                                                               | 1.1 MB  00:00:00     
(3/17): bind-chroot-9.9.4-61.el7.x86_64.rpm                                                                                                         |  87 kB  00:00:00     
(4/17): bind-devel-9.9.4-61.el7.x86_64.rpm                                                                                                          | 399 kB  00:00:00     
(5/17): bind-dyndb-ldap-11.1-4.el7.x86_64.rpm                                                                                                       | 122 kB  00:00:00     
(6/17): bind-libs-lite-9.9.4-61.el7.x86_64.rpm                                                                                                      | 734 kB  00:00:00     
(7/17): bind-libs-9.9.4-61.el7.x86_64.rpm                                                                                                           | 1.0 MB  00:00:00     
(8/17): bind-license-9.9.4-61.el7.noarch.rpm                                                                                                        |  85 kB  00:00:00     
(9/17): bind-lite-devel-9.9.4-61.el7.x86_64.rpm                                                                                                     | 308 kB  00:00:00     
(10/17): bind-pkcs11-9.9.4-61.el7.x86_64.rpm                                                                                                        | 298 kB  00:00:00     
(11/17): bind-pkcs11-devel-9.9.4-61.el7.x86_64.rpm                                                                                                  | 105 kB  00:00:00     
(12/17): bind-pkcs11-utils-9.9.4-61.el7.x86_64.rpm                                                                                                  | 198 kB  00:00:00     
(13/17): bind-pkcs11-libs-9.9.4-61.el7.x86_64.rpm                                                                                                   | 1.1 MB  00:00:00     
(14/17): bind-sdb-9.9.4-61.el7.x86_64.rpm                                                                                                           | 353 kB  00:00:00     
(15/17): bind-sdb-chroot-9.9.4-61.el7.x86_64.rpm                                                                                                    |  87 kB  00:00:00     
(16/17): bind-utils-9.9.4-61.el7.x86_64.rpm                                                                                                         | 204 kB  00:00:00     
(17/17): postgresql-libs-9.2.23-3.el7_4.x86_64.rpm                                                                                                  | 234 kB  00:00:00     
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                       12 MB/s | 8.1 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : GeoIP-1.5.0-11.el7.x86_64                                                                                                                              1/19 
  Updating   : 32:bind-license-9.9.4-61.el7.noarch                                                                                                                    2/19 
  Installing : 32:bind-libs-9.9.4-61.el7.x86_64                                                                                                                       3/19 
  Installing : 32:bind-9.9.4-61.el7.x86_64                                                                                                                            4/19 
  Installing : 32:bind-pkcs11-libs-9.9.4-61.el7.x86_64                                                                                                                5/19 
  Updating   : 32:bind-libs-lite-9.9.4-61.el7.x86_64                                                                                                                  6/19 
  Installing : postgresql-libs-9.2.23-3.el7_4.x86_64                                                                                                                  7/19 
  Installing : 32:bind-sdb-9.9.4-61.el7.x86_64                                                                                                                        8/19 
  Installing : 32:bind-sdb-chroot-9.9.4-61.el7.x86_64                                                                                                                 9/19 
  Installing : 32:bind-lite-devel-9.9.4-61.el7.x86_64                                                                                                                10/19 
  Installing : 32:bind-pkcs11-devel-9.9.4-61.el7.x86_64                                                                                                              11/19 
  Installing : 32:bind-pkcs11-9.9.4-61.el7.x86_64                                                                                                                    12/19 
  Installing : 32:bind-pkcs11-utils-9.9.4-61.el7.x86_64                                                                                                              13/19 
  Installing : 32:bind-chroot-9.9.4-61.el7.x86_64                                                                                                                    14/19 
  Installing : bind-dyndb-ldap-11.1-4.el7.x86_64                                                                                                                     15/19 
Enabling SELinux boolean named_write_master_zones
setsebool:  SELinux is disabled.
  Installing : 32:bind-devel-9.9.4-61.el7.x86_64                                                                                                                     16/19 
  Installing : 32:bind-utils-9.9.4-61.el7.x86_64                                                                                                                     17/19 
  Cleanup    : 32:bind-libs-lite-9.9.4-29.el7.x86_64                                                                                                                 18/19 
  Cleanup    : 32:bind-license-9.9.4-29.el7.noarch                                                                                                                   19/19 
  Verifying  : 32:bind-pkcs11-devel-9.9.4-61.el7.x86_64                                                                                                               1/19 
  Verifying  : 32:bind-pkcs11-9.9.4-61.el7.x86_64                                                                                                                     2/19 
  Verifying  : 32:bind-chroot-9.9.4-61.el7.x86_64                                                                                                                     3/19 
  Verifying  : 32:bind-sdb-9.9.4-61.el7.x86_64                                                                                                                        4/19 
  Verifying  : 32:bind-9.9.4-61.el7.x86_64                                                                                                                            5/19 
  Verifying  : 32:bind-devel-9.9.4-61.el7.x86_64                                                                                                                      6/19 
  Verifying  : GeoIP-1.5.0-11.el7.x86_64                                                                                                                              7/19 
  Verifying  : 32:bind-pkcs11-libs-9.9.4-61.el7.x86_64                                                                                                                8/19 
  Verifying  : 32:bind-lite-devel-9.9.4-61.el7.x86_64                                                                                                                 9/19 
  Verifying  : 32:bind-libs-9.9.4-61.el7.x86_64                                                                                                                      10/19 
  Verifying  : 32:bind-pkcs11-utils-9.9.4-61.el7.x86_64                                                                                                              11/19 
  Verifying  : 32:bind-libs-lite-9.9.4-61.el7.x86_64                                                                                                                 12/19 
  Verifying  : 32:bind-utils-9.9.4-61.el7.x86_64                                                                                                                     13/19 
  Verifying  : bind-dyndb-ldap-11.1-4.el7.x86_64                                                                                                                     14/19 
  Verifying  : 32:bind-license-9.9.4-61.el7.noarch                                                                                                                   15/19 
  Verifying  : postgresql-libs-9.2.23-3.el7_4.x86_64                                                                                                                 16/19 
  Verifying  : 32:bind-sdb-chroot-9.9.4-61.el7.x86_64                                                                                                                17/19 
  Verifying  : 32:bind-libs-lite-9.9.4-29.el7.x86_64                                                                                                                 18/19 
  Verifying  : 32:bind-license-9.9.4-29.el7.noarch                                                                                                                   19/19 

Installed:
  bind.x86_64 32:9.9.4-61.el7                bind-chroot.x86_64 32:9.9.4-61.el7          bind-devel.x86_64 32:9.9.4-61.el7     bind-dyndb-ldap.x86_64 0:11.1-4.el7        
  bind-libs.x86_64 32:9.9.4-61.el7           bind-lite-devel.x86_64 32:9.9.4-61.el7      bind-pkcs11.x86_64 32:9.9.4-61.el7    bind-pkcs11-devel.x86_64 32:9.9.4-61.el7   
  bind-pkcs11-libs.x86_64 32:9.9.4-61.el7    bind-pkcs11-utils.x86_64 32:9.9.4-61.el7    bind-sdb.x86_64 32:9.9.4-61.el7       bind-sdb-chroot.x86_64 32:9.9.4-61.el7     
  bind-utils.x86_64 32:9.9.4-61.el7         

Dependency Installed:
  GeoIP.x86_64 0:1.5.0-11.el7                                                    postgresql-libs.x86_64 0:9.2.23-3.el7_4                                                   

Updated:
  bind-libs-lite.x86_64 32:9.9.4-61.el7                                                 bind-license.noarch 32:9.9.4-61.el7                                                

Complete!
```

两个重要的配置文件：

/etc/named.rfc1912.zones ： 定义域

/var/named ：域的详细配置

之后记得要将文件的权限复制给named这个用户和组。


上面的配置可以连接到本地的定义的域名，但是无法ping通百度等。所以一般需要再配置文件中增加`forwarders`，当本地没有找到定义的域之后会在`forwarders`中定义的继续查找。打开forwarders之后记得修改`dnssec-validation no;`这样之后再ping 外网的域名就可以了。




