---
title: Harbor  镜像仓库安装与 Helm Chart 管理
copyright: true
date: 2022-12-26 21:02:35
tags: Harbor
categories: Harbor
---

# Harbor  镜像仓库安装与 Helm Chart 管理 

[Harbor](https://goharbor.io/) 是一款非常优秀的企业级开源镜像仓库管理器。Harbor 是基于 Docker Registry之上带有用户权限控制，镜像扫描，镜像签名的款工具。用起来很方便，我们公司目前在测试和生产都在使用。从去年 10 月到今天，除去一次 ceph 集群扩容，没有发生一次事故，一直运行很稳定；本文将记录一下我们的使用方式和遇到的一些问题解决方式。

## Harbor 资源准备

```
Harbor 1.7.6 版本
Redis 4.0
Postgres 10.6
Docker 1.13.1
docker-compose 1.18.0
Ceph 12

haproxy 机器 2 台
VIP 1 个
Harbor 主机 2 台
```

公司目前的 harbor 是基于 1.76 搭建的高可用方案，证书的问题，我们是公网使用第三方证书。内网使用自签名的证书；在我这里，我在 harbor 的搭建中采用的是内部 http，外部挂 F5。架构图如下：

![2020-04-26-21-15-43](/images/qiniu/2020-04-26-21-15-43.png)

生产上我们用 F5 替代 Haproxy。

我们在 Haproxy/F5 层将 https 转成 http 再转发给内部 harbor。
<!--more-->
### 下载 harbor 并进行配置

直接去github release 下载 1.7.6 的版本（现在最新为 1.10）。下载之后解压，修改里面`harbor.cfg`配置文件：

```cfg
## Configuration file of Harbor

#This attribute is for migrator to detect the version of the .cfg file, DO NOT MODIFY!
_version = 1.7.0
#The IP address or hostname to access admin UI and registry service.
#DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
#DO NOT comment out this line, modify the value of "hostname" directly, or the installation will fail.
hostname = harbor.chenzhijun.top

#The protocol for accessing the UI and token/notification service, by default it is http.
#It can be set to https if ssl is enabled on nginx.
ui_url_protocol = http

#Maximum number of job workers in job service
max_job_workers = 10

#Determine whether or not to generate certificate for the registry's token.
#If the value is on, the prepare script creates new root cert and private key
#for generating token to access the registry. If the value is off the default key/cert will be used.
#This flag also controls the creation of the notary signer's cert.
#这里我选择的是 off，开 notary 的话必须指定https，如果不开 notary 的话，可以不用管。
customize_crt = off

#The path of cert and key files for nginx, they are applied only the protocol is set to https
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key

#The path of secretkey storage
secretkey_path = /data

#Admiral's url, comment this attribute, or set its value to NA when Harbor is standalone
admiral_url = NA

#Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
log_rotate_count = 50
#Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes.
#If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G
#are all valid.
log_rotate_size = 200M

#Config http proxy for Clair, e.g. http://my.proxy.com:3128
#Clair doesn't need to connect to harbor internal components via http proxy.
http_proxy =
https_proxy =
no_proxy = 127.0.0.1,localhost,core,registry

#NOTES: The properties between BEGIN INITIAL PROPERTIES and END INITIAL PROPERTIES
#only take effect in the first boot, the subsequent changes of these properties
#should be performed on web ui

#************************BEGIN INITIAL PROPERTIES************************

#Email account settings for sending out password resetting emails.

#Email server uses the given username and password to authenticate on TLS connections to host and act as identity.
#Identity left blank to act as username.
email_identity =

email_server = smtp.mydomain.com
email_server_port = 25
email_username = sample_admin@mydomain.com
email_password = abc
email_from = admin <sample_admin@mydomain.com>
email_ssl = false
email_insecure = false

##The initial password of Harbor admin, only works for the first time when Harbor starts.
#It has no effect after the first launch of Harbor.
#Change the admin password from UI after launching Harbor.
harbor_admin_password = Harbor12345

##By default the auth mode is db_auth, i.e. the credentials are stored in a local database.
#Set it to ldap_auth if you want to verify a user's credentials against an LDAP server.
auth_mode = db_auth

#The url for an ldap endpoint.
ldap_url = ldaps://ldap.mydomain.com

#A user's DN who has the permission to search the LDAP/AD server.
#If your LDAP/AD server does not support anonymous search, you should configure this DN and ldap_search_pwd.
#ldap_searchdn = uid=searchuser,ou=people,dc=mydomain,dc=com

#the password of the ldap_searchdn
#ldap_search_pwd = password

#The base DN from which to look up a user in LDAP/AD
ldap_basedn = ou=people,dc=mydomain,dc=com

#Search filter for LDAP/AD, make sure the syntax of the filter is correct.
#ldap_filter = (objectClass=person)

# The attribute used in a search to match a user, it could be uid, cn, email, sAMAccountName or other attributes depending on your LDAP/AD
ldap_uid = uid

#the scope to search for users, 0-LDAP_SCOPE_BASE, 1-LDAP_SCOPE_ONELEVEL, 2-LDAP_SCOPE_SUBTREE
ldap_scope = 2

#Timeout (in seconds)  when connecting to an LDAP Server. The default value (and most reasonable) is 5 seconds.
ldap_timeout = 5

#Verify certificate from LDAP server
ldap_verify_cert = true

#The base dn from which to lookup a group in LDAP/AD
ldap_group_basedn = ou=group,dc=mydomain,dc=com

#filter to search LDAP/AD group
ldap_group_filter = objectclass=group

#The attribute used to name a LDAP/AD group, it could be cn, name
ldap_group_gid = cn

#The scope to search for ldap groups. 0-LDAP_SCOPE_BASE, 1-LDAP_SCOPE_ONELEVEL, 2-LDAP_SCOPE_SUBTREE
ldap_group_scope = 2

#Turn on or off the self-registration feature
# 是否允许自注册
self_registration = on

#The expiration time (in minute) of token created by token service, default is 30 minutes
token_expiration = 30

#The flag to control what users have permission to create projects
#The default value "everyone" allows everyone to creates a project.
#Set to "adminonly" so that only admin user can create project.
project_creation_restriction = everyone

#************************END INITIAL PROPERTIES************************

#######Harbor DB configuration section#######

#The address of the Harbor database. Only need to change when using external db.
#db的主机
db_host = 172.88.14.88

#The password for the root user of Harbor DB. Change this before any production use.
db_password = 123456

#The port of Harbor database host
db_port = 5432

#The user name of Harbor database
db_user = postgres

##### End of Harbor DB configuration#######

##########Redis server configuration.############

#Redis connection address
redis_host = 172.88.14.88

#Redis connection port
redis_port = 6379

#Redis connection password
redis_password =

#Redis connection db index
#db_index 1,2,3 is for registry, jobservice and chartmuseum.
#db_index 0 is for UI, it's unchangeable
redis_db_index = 1,2,3

########## End of Redis server configuration ############

##########Clair DB configuration############

#Clair DB host address. Only change it when using an exteral DB.

clair_db_host = 172.88.14.88
#The password of the Clair's postgres database. Only effective when Harbor is deployed with Clair.
#Please update it before deployment. Subsequent update will cause Clair's API server and Harbor unable to access Clair's database.
clair_db_password = 123456
#Clair DB connect port
clair_db_port = 5432
#Clair DB username
clair_db_username = postgres
#Clair default database
clair_db = postgres

#The interval of clair updaters, the unit is hour, set to 0 to disable the updaters.
clair_updaters_interval = 12

##########End of Clair DB configuration############

#The following attributes only need to be set when auth mode is uaa_auth
uaa_endpoint = uaa.mydomain.org
uaa_clientid = id
uaa_clientsecret = secret
uaa_verify_cert = true
uaa_ca_cert = /path/to/ca.pem


### Harbor Storage settings ###
#Please be aware that the following storage settings will be applied to both docker registry and helm chart repository.
#registry_storage_provider can be: filesystem, s3, gcs, azure, etc.
#registry_storage_provider_name = filesystem
registry_storage_provider_name = s3

#registry_storage_provider_config is a comma separated "key: value" pairs, e.g. "key1: value, key2: value2".
#To avoid duplicated configurations, both docker registry and chart repository follow the same storage configuration specifications of docker registry.
#Refer to https://docs.docker.com/registry/configuration/#storage for all available configuration.

#registry_storage_provider_config =
#配置ceph 存储
registry_storage_provider_config = bucket: f4gkewos23fdsf8fnfhG, region: default, accesskey: QaidfneuhgfE2dife, secretkey: qyIJDFNGIDNDKF8f2r3G5QSw, regionendpoint: http://ceph.chenzhijun.top, rootdirectory: /harbor-registry/di

#registry_custom_ca_bundle is the path to the custom root ca certificate, which will be injected into the truststore
#of registry's and chart repository's containers.  This is usually needed when the user hosts a internal storage with self signed certificate.
registry_custom_ca_bundle =

#If reload_config=true, all settings which present in harbor.cfg take effect after prepare and restart harbor, it overwrites exsiting settings.
#reload_config=true
#Regular expression to match skipped environment variables
#skip_reload_env_pattern=(^EMAIL.*)|(^LDAP.*)
```

配置好 cfg 文件后，我们开始我们的操作。

0. 申请或安装 ceph，ceph 版本为 12；

1. 安装 pg 数据库(生产应为高可用)：

`docker run --name pg -e POSTGRES_PASSWORD=123456 -p 5432:5432 -d postgres:10.6`

2. 安装 redis（生产因为高可用）：

`docker run --name redis -p 6379:6379 -d redis`

3. 安装 harbor：

`./install.sh --with-clair --with-chartmuseum`

这里除了 notaty 必须要有个 https 外，其他的你都可以装。clair 是镜像扫描工具；chartmuseum 是 helm chart ；
你需要在两台机器都启动，并且保持cfg 文件一致。

4. 配置 haproxy(F5)

如果在 F5 做 https 转 http，那haproxy 就正常启动并将后段指向 harbor 地址就可以了。如果是让 https 做证书认证，那么就按照配置就好。我这边有在 F5 做解证书，贴一份 haproxy 的配置吧`haproxy.cfg`：

```conf
global
  daemon
  log  127.0.0.1 local0 info
  maxconn  20000
  pidfile  /app/haproxy/run/haproxy.pid
  stats  socket /app/haproxy/lib/haproxy/stats
  tune.bufsize  131072
  user mwop
  group mwop
  tune.ssl.default-dh-param 2048

defaults
  log  global
  maxconn  10000
  mode  http
  option  dontlog-normal
  option  http-server-close
  retries  3
  #stats  enable
  timeout  http-request 100s
  timeout  queue 1m
  timeout  connect 10s
  timeout  client 1m
  timeout  server 30m
  timeout  check 10s

listen Stats
  bind 0.0.0.0:10000
  mode http
  stats enable
  stats uri /
  stats refresh 5s
  stats show-node
  stats show-legends
  stats hide-version

listen app1
    bind :80
    balance     roundrobin
    mode tcp
    server s1  172.0.0.1:80   weight 1
    server s2  172.0.0.2:80   weight 1
    
listen app2
    bind :443 
    balance     roundrobin
    mode tcp
    server s1  172.0.0.1:443   weight 1
    server s2  172.0.0.2:443   weight 1
```

访问`harbor.chenzhijun.top`就可以了。

![2020-04-26-21-43-00](/images/qiniu/2020-04-26-21-43-00.png)

### https 问题 1 : docker login failed
安装完之后，你可能会遇到一些问题，比如在你直接 docker login 出现:

```
Error response from daemon: Get https://harbor.chenzhijun.top/v2/: Get http://harbor.chenzhijun.top/service/token?account=admin&client_id=docker&offline_token=true&service=harbor-registry: dial tcp 100.77.53.130:80: getsockopt: connection refused
```

你在页面登陆可以，当是你用 docker login却有问题。你可以`curl -X GET -I "https://harbor.chenzhijun.top/v2/"`看一下是否返回了一个 http 的请求。如果是的话，修改一个参数：`sed -i "s/realm: http/realm: https/g" common/config/registry/config.yml`;在`common/config/registry/config.yml`中修改返回值为 https：
```yaml
auth:
  token:
    issuer: harbor-token-issuer
    realm: https://harbor.chenzhijun.top/service/token #就是这里要返回 https 而不是 http
    rootcertbundle: /etc/registry/root.crt
    service: harbor-registry
```
这个地方改不是特别好改，我是在`install.sh`在 docker-compose启动之前加入的这个 sed 命令；

![2020-04-26-22-00-25](/images/qiniu/2020-04-26-22-00-25.png)

然后再 curl 一下看是不是返回 https 的结果了。如果是就可以了。


### https 问题2 : docker push 出现 unkown blob

因为我们的架构是：域名 --> dns解析到 --> F5 VIP --> haproxy --> Harbor docker-compose;

在这个过程中，出现了一个问题。域名到到 F5是 https，而后面的 Harbor 接到的 http；所以在 docker login 成功。在 docker push 的时候出现了问题 `unkown blob`；这个问题的原因就是在反向代理中 harbor 的 nginx 代理用的还是 https 方式：[Harbor Troubleshooting](https://github.com/goharbor/harbor/blob/release-1.7.0/docs/installation_guide.md#Troubleshooting)。修改`common/templates/nginx/nginx.http.conf`中的`proxy_set_header X-Forwarded-Proto $scheme;`把这行注释掉就可以了。

另外就是要注意 pg 的权限问题。

### https 问题3: docker push 出现 unauthorized: authentication required

这是一个很诡异的问题，找了我两个小时；现象是harbor 的页面能登陆，并且页面一切功能正常；直接使用 docker login 也是没有问题，但是在 push 镜像的时候出现，unauthorized: authentication required;明明已经登陆了，也没有问题，但是不知道为啥还是会报错。这个问题在我这里是由于 haproxy 的配置出现的问题。harbor push 镜像的过程是从 harbor 服务获取一个 token，然后再去访问 registry 的存储来存储数据，如果是是本地存储，那么会访问 80 端口。我的配置中将 haproxy 的 80 端口转了 redirect，所以导致认证失败。附上一段在 haproxy 段做证书解析而后端 harbor 为 http 的 haproxy 的服务配置：

```conf
frontend harbor
  bind *:80
  bind *:443 ssl crt /data/harbor/harbor/chenzhijun.top.pem
  reqadd X-Forwarded-Proto:\ https
  default_backend  harbor-backend

backend harbor-backend
  balance  source
  mode  http
  timeout  client 3h
  timeout  server 3h
  server server1 10.1.1.2:80  check inter 2000 fall 3
```

如果还是有问题，可以在`/var/log/harbor`目录下查看相关的 log 日志。也可以查看系统日志：`/var/log/message`来定位问题。

我们也安装了 `/install.sh --with-clair --with-chartmuseum` clair 和 chartmuseum，这两个工具，clair 的主要目的是扫描 CVE 漏洞，所以是不是需要链接一下外网更新一下；chartmuseum 是一个 helm chart 管理器;

![2020-04-26-22-15-03](/images/qiniu/2020-04-26-22-15-03.png)

![2020-04-26-22-16-24](/images/qiniu/2020-04-26-22-16-24.png)

## 升级问题

最近尝试在本地进行了一次 1.7 升级到 1.8，其实还好啊，就是 cfg 文件变成了 yml 文件，其他的还是一样。并且有迁移工具：
https://github.com/goharbor/harbor/blob/release-1.8.0/docs/migration_guide.md

迁移工具使用方式：

`docker run -it --rm -v /data/harbor/harbor/harbor.cfg:/harbor-migration/harbor-cfg/harbor.cfg -v /data/harbor/harbor/harbor.yml:/harbor-migration/harbor-cfg-out/harbor.yml harbor.uat.x.com/goharbor/harbor-migrator:v1.8.3 --cfg up`

然后就停掉原来的 1.7 使用新的 1.8 的安装包启动就可以了。

迁移前一定要进行数据备份，一定要备份。


## helm 使用

helm 在 v3 之后（我只用了 v3）其实特别好用。在 github 上下载 helm 的安装包，然后把 helm 放到 /usr/local/bin 下面就可以直接执行 helm 命令了。当然前提是要本机有 kubectl，并且本地有 kubeconfig 文件`~/.kube/config`。这样我们就能愉快的使用 helm 了。那我们怎么使用 harbor 来管理我们的 helm chart 了？

`helm repo add --username readonly --password Read2019 myharbor http://harbor.chenzhijun.top/chartrepo/helm-repo`

`http://harbor.chenzhijun.top/chartrepo/` 这一段是固定的，`helm-repo`是 harbor 中的 project 名。然后你将一个 chart 包导入：

![2020-04-26-22-25-53](/images/qiniu/2020-04-26-22-25-53.png)

`helm search repo myharbor`

`helm repo update`

`helm install myharbor/consul --gernerate-name`

好了，就是这么简单粗暴。

