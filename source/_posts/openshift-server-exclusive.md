---
title: Openshift 专属机方案
copyright: true
date: 2022-02-17 20:59:00
tags: Kubernetes
categories: Kubernetes
---

# Openshift 专属机方案

<!--more--> 
## 专属机方案-Kubernetes(一）

1. 给所有主机新增 role:  [`node-role.kubernetes.io/{xxxxx}:](http://node-role.kubernetes.io/%7Bxxxxx%7D:) "true"`;这样在 oc get nodes 的时候就会多出一个 `xxxxx` 的角色；
2. 给相关的主机打上污点：`oc adm nodes $i com.cmft.exclusive=xxxxx:NoSchedule` / `kubectl taint node com.cmft.exclusive=xxxxx:NoSchedule`
3. 让所有的业务 pod 增加主机调度和容忍，`nodeSelector`,`tolerations`：

```bash
spec:
  containers:
  - image: harbor.uat.chenzhijun.top/base/nginx:latest
    name: nginx
    resources: {}
  nodeSelector: 
    node-role.kubernetes.io/compute: "true"
    node-role.kubernetes.io/xxxxx: "true"
  tolerations: 
  - key: "com.cmft.exclusive"
    operator: "Equal"
    value: "xxxxx"
    effect: "NoSchedule"
```

## 专属机方案-Openshift（二）

1.  选定主机并且将主机的 label 进行修改，将 label 名改为 [`node-role.kubernetes.io](node-role.kubernetes.io/shimo)/xxx=true`，这样在 `oc get nodes` 时可以看到 `xxx` 的角色的主机；
2.  project（namespace）在创建的时候可以使用`oc adm new-project demo --node-selector='node-role.kubernetes.io/xxx=true'` 这样默认 project 会只调度到selector 的主机上。
3. 如果是已存在的项目：`oc patch namespace demo -p '{"metadata":{"annotations":{"openshift.io/node-selector":"node-role.kubernetes.io/xxx=true"}}}'` 可以进行 patch 操作；如果有多个 label 可以进行：`oc patch namespace demo -p '{"metadata":{"annotations":{"openshift.io/node-selector":"node-role.kubernetes.io/xxx1=true,node-role.kubernetes.io/xxx2=true"}}}'`
4. 如果需要设置全局默认调度可以修改 master 配置文件：`/etc/origin/master/master-config.yaml`  `projectConfig.defaultNodeSelector` 然后重启 api 和 controller：`master-restart api`&&`master-restart api`