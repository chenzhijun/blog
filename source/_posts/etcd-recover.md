---
title: ETCD 主机恢复
copyright: true
date: 2021-12-19 23:01:09
tags: Kubernetes
categories: Kubernetes
---

# ETCD 主机恢复

背景：在没有master 备份的情况下， 集群中有一个master 节点被直接重装系统；该节点非 ETCD master 节点。所以集群还是处于可用状态。但是 master 由之前的三节点变为 2 节点；
<!--more--> 
openshift  版本：v170

1. ETCD 备份
    
    ```bash
    #etcd master 节点上操作
    yum install -y etcd
    systemctl disable etcd.service
    systemctl mask etcd.service
    export ETCDCTL_API=3
    mkdir -p /backup/etcd-config-$(date +%Y%m%d)/
    cp -R /etc/etcd/ /backup/etcd-config-$(date +%Y%m%d)/
    
    oc get nodes -o wide|grep master |awk '{print $6":2379"}'|xargs|tr ' ' ','
    ETCD_ENDPOINTS=$(oc get nodes -o wide|grep master |awk '{print $6":2379"}'|xargs|tr ' ' ',')
    
    etcdctl --cert /etc/etcd/peer.crt --key /etc/etcd/peer.key --cacert /etc/etcd/ca.crt --endpoints=$ETCD_ENDPOINTS snapshot save /var/lib/etcd/snapshot.db
    
    #要先删掉无效 node,不然redeploy 证书的时候，会卡在 remove console 的步骤
    #oc delete nodes [UNKONWN_NODE]
    oc get nodes|grep master|grep NotReady|awk '{print $1}'|xargs -i oc delete nodes {}
    
    cnsz92vl12816.cmftdc.cn
    ```
    
2. 重新部署 ETCD-CA

```bash
export TOKEN=eyJhbGciOiJIUzUxMiJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.5DWDErsUzcBYK-KD_j5tjemwPIrLMU3Xle5lDaoj-3HkYBeMQ2WTvF7wvkIj4Kint_XABxT7MgInCp9Z-gklyw

#0.恢复 ETCD 的 CA， 然后 certificate， 然后 master ca， master certificate；
#进入容器，增加 redeploy-etcd-ca.yml
#重新部署 ca
cp add_new_nodes.yml redeploy-etcd-ca.yml
---
- import_playbook: openshift-etcd/redeploy-ca.yml

#退出容器，执行部署 ca，这里还没有增加主机
curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{}' https://localhost:5001/api/v1/playbooks/redeploy-etcd-ca.yml -X POST

UUID=xxxx
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/jobs/$UUID/stdout -X GET
```

3. 重新部署 ETCD 证书

```bash
#复制 etcd 证书更新 playbook
cp add_new_nodes.yml redeploy-etcd-certificates.yml
---
- import_playbook: openshift-etcd/redeploy-certificates.yml

#重新部署 etcd 证书
curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{}' https://localhost:5001/api/v1/playbooks/redeploy-etcd-certificates.yml -X POST

cp add_new_nodes.yml redeploy-master-ca.yml
---
- import_playbook: openshift-master/redeploy-openshift-ca.yml

#重新部署 master 证书, 不执行，master 有
#curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{}' https://localhost:5001/api/v1/playbooks/redeploy-master-ca.yml -X POST

==>
curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{}' https://localhost:5001/api/v1/playbooks/redeploy-certificates.yml -X POST

UUID=xxxx
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/jobs/$UUID/stdout -X GET
```

4. 增加 master 主机

```bash
#重新部署一遍 master 证书，不然后面的Wait for /apis/metrics.k8s.io/v1beta1 when registered 会出现异常,最好重新部署一遍证书
ansible-playbook -i ./inventory project/openshift-master/redeploy-openshift-ca.yml

##增加主机
export ETCD_NODES=cnsz92vl12816.cmftdc.cn

###1. 增加 hosts 配置
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/groups/new_nodes -X POST
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/groups/new_masters -X POST
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/hosts/$ETCD_NODES/groups/new_nodes -X POST 
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/hosts/$ETCD_NODES/groups/new_masters -X POST 
curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{"openshift_node_group_name": "node-config-master"}' https://localhost:5001/api/v1/hostvars/$ETCD_NODES/groups/new_nodes?type=inventory -X POST
curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{"openshift_node_group_name": "node-config-master"}' https://localhost:5001/api/v1/hostvars/$ETCD_NODES/groups/new_masters?type=inventory -X POST
```

5. 恢复 master pod 组件

```bash
###恢复 master
进入到 ansible-runner-service 容器中，
cd /root/ansible-runner-service/cmg-ocp/project
cp add_new_nodes.yml add_new_masters.yml
#修改里面的内容为 openshfit-master
---
- import_playbook: openshift-master/scaleup.yml

curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{}' https://localhost:5001/api/v1/playbooks/add_new_masters.yml -X POST

UUID=xxxx
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/jobs/$UUID/stdout -X GET
```

```bash

export ETCD_NODES=cnsz92vl12816.cmftdc.cn

curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/groups/new_etcd -X POST
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/hosts/$ETCD_NODES/groups/new_etcd -X POST

curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{"openshift_node_group_name": "node-config-master"}' https://localhost:5001/api/v1/hostvars/$ETCD_NODES/groups/new_etcd?type=inventory -X POST

#恢复 etcd
#需要将 hosts 文件里面的 new_master,new_nodes 改到正确的位置
vi /etc/ansible-runner/inventory/hosts

OSEv3:
  children:
    etcd:
      hosts:
        cnsz92vl10440.cmftdc.cn: null
        cnsz92vl10441.cmftdc.cn: null
    masters:
      hosts:
        cnsz92vl10442.cmftdc.cn: null
        cnsz92vl10440.cmftdc.cn: null
        cnsz92vl10441.cmftdc.cn: null
    new_etcd:
      hosts:
        cnsz92vl10442.cmftdc.cn: null
    nodes:
      hosts:
        cnsz92vl10442.cmftdc.cn:
          openshift_node_group_name: node-config-master
        cnsz92vl10440.cmftdc.cn:
          openshift_node_group_name: node-config-master
        cnsz92vl10441.cmftdc.cn:
          openshift_node_group_name: node-config-master
        cnsz92vl10443.cmftdc.cn:
          openshift_node_group_name: node-config-infra
        cnsz92vl10445.cmftdc.cn:
          openshift_node_group_name: node-config-infra
        cnsz92vl10448.cmftdc.cn:
          openshift_node_group_name: node-config-compute
        cnsz92vl11127.cmftdc.cn:
          openshift_node_group_name: node-config-infra

```

6. 检查 hosts 文件，确保之前的 new_masters 配置已删除

7. 确认 ETCD 中失败节点删除

```bash

100.69.137.3:2379,100.69.137.4:2379,100.69.137.5:2379

#删除集群中原来的老 etcd 节点

export ETCDCTL_API=3
#查询etcd member list
etcdctl --cert /etc/etcd/peer.crt --key /etc/etcd/peer.key --cacert /etc/etcd/ca.crt --endpoints="100.69.137.3:2379,100.69.137.4:2379,100.69.137.5:2379" member list
#查询集群状态详情
etcdctl --cert /etc/etcd/peer.crt --key /etc/etcd/peer.key --cacert /etc/etcd/ca.crt --endpoints="100.69.137.3:2379, has been 100.69.137.4:2379,100.69.137.5:2379" --write-out=table endpoint status
#删除失败节点
etcdctl --cert /etc/etcd/peer.crt --key /etc/etcd/peer.key --cacert /etc/etcd/ca.crt --endpoints="100.69.137.3:2379,100.69.137.4:2379,100.69.137.5:2379" member remove ID

#命令执行 example
etcdctl --cert /etc/etcd/peer.crt --key /etc/etcd/peer.key --cacert /etc/etcd/ca.crt --endpoints="100.69.137.3:2379,100.69.137.4:2379,100.69.137.5:2379" --write-out=table endpoint status
Failed to get the status of endpoint 100.75.46.77:2379 (context deadline exceeded)
+-------------------+------------------+---------+---------+-----------+-----------+------------+
|     ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+-------------------+------------------+---------+---------+-----------+-----------+------------+
| 100.75.46.76:2379 | 29ace8a008187a61 |  3.2.22 |   26 MB |     false |        15 |      74501 |
| 100.75.46.78:2379 | 189fb2b310767596 |  3.2.22 |   26 MB |      true |        15 |      74516 |
+-------------------+------------------+---------+---------+-----------+-----------+------------+
[root@cnsz92vl10441 ~]# etcdctl --cert /etc/etcd/peer.crt --key /etc/etcd/peer.key --cacert /etc/etcd/ca.crt --endpoints="100.75.46.76:2379,100.75.46.77:2379,100.75.46.78:2379" member list
189fb2b310767596, started, cnsz92vl10441, https://100.75.46.78:2380, https://100.75.46.78:2379
29ace8a008187a61, started, cnsz92vl10440, https://100.75.46.76:2380, https://100.75.46.76:2379
b25d6fd89ab25bb3, started, cnsz92vl10442, https://100.75.46.77:2380, https://100.75.46.77:2379
[root@cnsz92vl10441 ~]# etcdctl --cert /etc/etcd/peer.crt --key /etc/etcd/peer.key --cacert /etc/etcd/ca.crt --endpoints="100.75.46.76:2379,100.75.46.77:2379,100.75.46.78:2379" member remove b25d6fd89ab25bb3
Member b25d6fd89ab25bb3 removed from cluster 4547a2deaf4fef8e
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b67bcee1-ec5d-4ed1-bd39-181a74cde946/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b67bcee1-ec5d-4ed1-bd39-181a74cde946/Untitled.png)

8. 恢复 ETCD

```bash
#增加 new_etcd 分组 . 只要有 new_etcd 这个组，不要 new_masters, new_nodes
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/groups/new_etcd -X POST

curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/hosts/$ETCD_NODES/groups/new_etcd -X POST 

cd /root/ansible-runner-service/cmg-ocp/project
cp add_new_nodes.yml add_new_etcd.yml
#修改里面的内容为 openshfit-master
---
- import_playbook: openshift-etcd/scaleup.yml

#删除原来集群中失败的 etcd 节点;会拉取新的 etcd 镜像，保持镜像仓库镜像只有一个版本；
curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{}' https://localhost:5001/api/v1/playbooks/add_new_etcd.yml -X POST
UUID=xxxx
curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/jobs/$UUID/stdout -X GET

```

9. 验证

```bash
etcdctl --cert /etc/etcd/peer.crt --key /etc/etcd/peer.key --cacert /etc/etcd/ca.crt --endpoints="100.75.46.76:2379,100.75.46.77:2379,100.75.46.78:2379" --write-out=table endpoint status

oc get pods -n kube-system
```

附录：

1. bootstrap 常用参数命令
    
    ```bash
    #TOKEN 导入
    export TOKEN=eyJhbGciOiJIUzUxMiJ9.eyJ1c2VybmFtZSI6ImFkbWluIn0.5DWDErsUzcBYK-KD_j5tjemwPIrLMU3Xle5lDaoj-3HkYBeMQ2WTvF7wvkIj4Kint_XABxT7MgInCp9Z-gklyw
    
    #部署集群
    curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{}' https://localhost:5001/api/v1/playbooks/deploy_cluster.yml -X POST
    
    #卸载集群
    curl -k -i -H "Content-Type: application/json" -H "Authorization: $TOKEN" --data '{}' https://localhost:5001/api/v1/playbooks/uninstall.yml -X POST
    
    ##删除命令
    curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/groups/new_etcd -X DELETE
    
    curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/groups/new_masters -X DELETE
    
    curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/groups/new_nodes -X DELETE
    
    #实时日志
    curl -k -i -H "Authorization: $TOKEN" https://localhost:5001/api/v1/jobs/$UUID/stdout -X GET
    ```
    
2. 卡在某个 ansible 脚本
    
    ```bash
    fatal: [cnsz92vl10442.cmftdc.cn -> cnsz92vl10441.cmftdc.cn]: FAILED! => {"changed": true, "cmd": ["oc", "adm", "create-api-client-config", "--certificate-authority=/etc/origin/master/ca.crt", "--client-dir=/tmp/openshift-ansible-O3mFaX", "--groups=system:masters,system:openshift-master", "--master=https://cnsz92vl10441:8443", "--public-master=https://cnsz92vl10441:8443", "--signer-cert=/etc/origin/master/ca.crt", "--signer-key=/etc/origin/master/ca.key", "--signer-serial=/etc/origin/master/ca.serial.txt", "--user=system:openshift-master", "--basename=openshift-master", "--expire-days=730"], "delta": "0:00:00.193549", "end": "2021-01-11 15:59:54.383491", "msg": "non-zero return code", "rc": 1, "start": "2021-01-11 15:59:54.189942", "stderr": "error: --signer-serial, \"/etc/origin/master/ca.serial.txt\" must be a valid file", "stderr_lines": ["error: --signer-serial, \"/etc/origin/master/ca.serial.txt\" must be a valid file"], "stdout": "", "stdout_lines": []}
    
    看下是否 node 没有从集群中删除
    ```
    
3.  错误
    
    ```bash
    TASK [openshift_ca : Install the base package for admin tooling] ***************
    FAILED - RETRYING: Install the base package for admin tooling (3 retries left).
    FAILED - RETRYING: Install the base package for admin tooling (2 retries left).
    FAILED - RETRYING: Install the base package for admin tooling (1 retries left).
    fatal: [cnsz92vl10442.cmftdc.cn -> cnsz92vl10441.cmftdc.cn]: FAILED! => {"attempts": 3, "changed": false, "msg": "No package matching 'atomic-openshift-3.11.170' found available, installed or updated", "rc": 126, "results": ["No package matching 'atomic-openshift-3.11.170' found available, installed or updated"]}
    
    ansible masters -i hosts -m shell -a "yum clean all"
    
    #有可能少包，那就需要手动装
    scp atomic-openshift-3.11.170-1.git.0.00cac56.el7.x86_64.rpm cnsz92vl10441:/tmp/
    
    rpm -Uvh atomic-openshift-3.11.170-1.git.0.00cac56.el7.x86_64.rpm
    ```
    
    ```bash
    ansible-playbook -i ./inventory ./project/openshift-master/redeploy-openshift-ca.yml
    
    ansible-playbook -i ./inventory ./project/redeploy-certificates.yml
    
    ansible-playbook -i ./inventory ./project/openshift-master/redeploy-certificates.yml
    
    osm_etcd_image=harbor.uat.cmft.com/rhel7/etcd:3.2.22
    openshift_pkg_version=-3.11.170
    
    openshift_is_atomic=true
    ```