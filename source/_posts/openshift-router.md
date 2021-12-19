---
title: OpenShift 专用 Router 实操
copyright: true
date: 2021-02-27 22:53:35
tags: Kubernetes
categories: Kubernetes
---

# Openshift 专用 Router 实操

由于某些服务在实际业务场景中非常重要，或者为了与其它业务进行入口区分，因此会产生专属 router（Ingress）的需求，在 openshift 的实际操作可以这样：

<!--more--> 

```shell

#确保 sa 为 router 名字的有scc hostnetwork 权限
oc adm policy add-scc-to-user hostnetwork -z router

#创建一个 router ，指定 router-iot 为名字
oc adm router router-iot --images='harbor.uat.cmft.com/openshift3/ose-haproxy-router:v3.11.170' --selector='node-role.kubernetes.io/iot-router=true' --labels='router=iot'

#设置 router 的 ns 标签
oc set env dc/router-iot NAMESPACE_LABELS="router=iot”

#设置 ns 标签
oc label namespace demo “router=iot”

#设置某个 ns 下的 router 必须有 cluster-reader 权限
oc adm policy add-cluster-role-to-user cluster-reader system:serviceaccount:demo:router

#https://docs.openshift.com/container-platform/3.11/admin_guide/manage_rbac.html
oc adm policy add-role-to-user cluster-reader system:serviceaccount:demo:router

#修改 router 监听端口
oc adm router --replicas=0 --ports='10080:10080,10443:10443' 
oc set env dc/router ROUTER_SERVICE_HTTP_PORT=10080 ROUTER_SERVICE_HTTPS_PORT=10443
oc scale dc/router --replicas=1

iptables -A OS_FIREWALL_ALLOW -p tcp --dport 10080 -j ACCEPT
iptables -A OS_FIREWALL_ALLOW -p tcp --dport 10443 -j ACCEPT

#暴露一个 route
oc expose service myservice --hostname=owner.example.test

#允许泛域名
oc set env dc/router ROUTER_ALLOW_WILDCARD_ROUTES=true

#修改泛域名子域
oc adm router --force-subdomain='${name}-${namespace}.apps.example.com'

```
