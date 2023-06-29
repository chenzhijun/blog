---
title: 容器网络 - CNI网络插件
copyright: true
date: 2023-04-15 22:06:38
tags: Kubernetes
categories: Kubernetes
---

# 容器网络 - CNI网络插件

上一篇里面，我们讲了容器网络，跨主机网络通信(flannel的udp和vxlan)；这里我们再了解一下cni它是怎么实现的，然后再说一下另一种网络模型HostGW。

跨主机通信中，不管是udp模式，还是vxlan模式，他们都有一个相同点，用户的容器连在了docker0网桥上。网络插件在主机上创建一个特殊的设备(UDP模式创建的TUN设备，VXLAN模式创建VTEP设备)；docker0与这个设备配合，通过ip转发(路由表)协作。

网络插件真正要做的事情，是把不同主机上的特殊设备通过某种方法连通，达到容器跨主机网络通信的目的。kubernetes 通过 CNI 的接口，单独维护了一个网桥代替docker0，名字叫CNI网络，宿主机上的设备默认名称为cni0。
<!--more--> 
![vxlan 模式图](/images/qiniu/image-20230629162232298.png)

在主机上可以使用`route -n `查看主机路由表。

为啥kubernetes不直接用docker0网桥呢？一个是它没有使用Docker的网络模型(CNM)，所以它不具备配置docker0网桥的能力。另一个这一环节与如何配置pod，也就是infra容器的networkd namespace密切相关。

Kubernetes 创建pod的第一步 ，是创建并启动一个infra容器，用来hold住pod的network namespace，所以cni的设计思想，就是在kubernetes在infra容器之后，可以直接调用cni网络插件，为这个infra容器的network namespace配置符合预期的网络栈。

> 一个network namespace 网络栈包括：网卡(Network Interface)、回环设备(Loopback Device)、路由表(Routing Table)、iptables规则。

那这个网络栈配置工作如何完成？首先我们看一下，cni插件安装了哪些东西。在/opt/cni/bin目录下，看一下CNI插件所需的基础可执行文件。

```

$ ls -al /opt/cni/bin/
total 73088
-rwxr-xr-x 1 root root  3890407 Jun 17   bridge
-rwxr-xr-x 1 root root  9921982 Jun 17   dhcp
-rwxr-xr-x 1 root root  2814104 Jun 17   flannel
-rwxr-xr-x 1 root root  2991965 Jun 17   host-local
-rwxr-xr-x 1 root root  3475802 Jun 17   ipvlan
-rwxr-xr-x 1 root root  3026388 Jun 17   loopback
-rwxr-xr-x 1 root root  3520724 Jun 17   macvlan
-rwxr-xr-x 1 root root  3470464 Jun 17   portmap
-rwxr-xr-x 1 root root  3877986 Jun 17   ptp
-rwxr-xr-x 1 root root  2605279 Jun 17   sample
-rwxr-xr-x 1 root root  2808402 Jun 17   tuning
-rwxr-xr-x 1 root root  3475750 Jun 17   vlan
```

CNI的基础可执行文件，可以分为三类：

第一类，叫做Main插件，它是用来创建具体网络设备的二进制文件。比如，bridge(网桥设备)、ipvlan、loopback(lo设备)、macvlan、ptp(Veth Pair设备)，以及vlan。

第二类，叫做IPAM(IP address Management)插件，它是负责分配IP地址的二进制文件。比如，dhcp，这个文件会想DHCP服务器发起请求；host-local，则会使用预先配置的IP地址段来进行分配。

第三类，是有CNI社区维护的内置CNI插件。比如：flannel，就是转么为Flannel项目提供的CNI插件；tuning，是一个通过sysctl调整网络设备参数的二进制文件；portmap，是一个通过iptables配置端口映射的二进制文件；bandwidth，是一个使用Token Bucket Filter来进行限流的二进制文件。

从这些二进制文件中，可以看到实现一个kubernetes用的容器网络方案，其实需要做两部分工作：

1. 实现这个网络方案本身。这一部分需要编写的，其实就是flanneld进程里的主要逻辑。比如，创建和配置flannel.1 设备、配置宿主机路由、配置arp和fdb表里的信息等等。
2. 实现该网络方案对应的cni插件，这一部分主要需要做的，就是配置infra容器里面的网络栈，并把它连接到cni网桥上。

需要注意的是， kubelet不会直接调用cni插件， 调用cni是在cri(Container Runtime Interface，容器运行时接口)里面去实现的。比如docker，它的cri叫做dockershim。kubernetes本身不支持多个cni配置，如果在CNI配置目录(/etc/cni/net.d)里面放了多个cni配置，cri只会加载按字母排序的第一个插件。不过另一方面，CNI允许在CNI配置文件里面通过plugins字段，定义多个插件协作：

```

$ cat /etc/cni/net.d/10-flannel.conflist 
{
  "name": "cbr0",
  "plugins": [
    {
      "type": "flannel",
      "delegate": {
        "hairpinMode": true,
        "isDefaultGateway": true
      }
    },
    {
      "type": "portmap",
      "capabilities": {
        "portMappings": true
      }
    }
  ]
}
```

上面这个例子，就是flannel和pormap这两个插件协作。dockershim(cri)在家CNI配置时，会把第一个flannel插件，设置为默认插件。在后面的执行过程中，flannel和pormap插件会按照定义顺序被调用，从而依次完成"配置容器网络"和"配置端口映射"这两部操作。

我们现在整体看一下cni插件的工作原理。

当kubelet组件需要创建pod的时候，它第一个创建的一定是infra容器。所以这一步dockershim (CRI) 会先调用Docker api 创建并启动infra容器，紧接着，回执行一个叫做SetUpPod的方法，这个方法的作用就是：为CNI插件准备参数，然后调用 CNI 插件(/opt/cni/bin/flannel) 为 Infra 容器配置网络。flannel插件，调用它需要两部分组成：

第一部分：由dockershim设置的一组CNI环境变量。

其中最重要的环境变量参数叫做：CNI_COMMAND。它的取值只有两种：ADD和DEL。这两个操作也是CNI插件唯一需要实现的两个方法。其中ADD的含义是，把容器添加到CNI网络里；DEL的含义是：把容器从CNI网络里移除掉。对于网桥类型的CNI插件来说，这两个操作以为着把容器以VethPair的方式"插"到CNI网络或者从网桥上"拔"掉。

着重看一下CNI的ADD操作。ADD操作需要的参数包括：容器里网卡的名字eth0(CNI_IFNAME)、Pod的network Namespace文件的路径(CNI_NETNS)、容器的ID(CNI_CONTAINERID)等。

第二部分：则是dockershim从cni配置文件里加载到的、默认插件的配置信息。

这个配置信息在CNI中被叫做Network Configuration。dockershim会把NetworkConfiguration以JSON数据的格式，通过标准输入stdin的方式传递给CNI插件。CNI插件里面可能会根据配置文件又去调用其它插件，然后来完成整个网络的配置。

当CNI插件执行完ADD操作后，CNI插件会把容器的IP地址返回给dockershim，然后被kubelet添加到Pod的status字段。至此，整个流程结束。这就是网络类型的CNI插件整个操作流程。


