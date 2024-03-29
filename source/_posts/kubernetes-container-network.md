---
title: 容器的网络基础
copyright: true
date: 2023-02-11 21:48:52
tags: Kubernetes
categories: Kubernetes
---

# 容器的网络基础

容器的网络基础是主机的网络，在理解容器的网络前我们先来看一下主机的网络。

> 想象一个场景，你在湖南，你的爱人在湖北，有一天，你们两觉得恋爱的时间差不多了，该结婚了。结婚之前是不是要先见一下家长。但是湖南和湖北之间隔着长江。那你现在要去湖北的话，怎么办呢？你会想，我有车，只需要开车过“长江大桥”去对方父母家就行。

换一下，现在你有两台电脑，你想让两台电脑能够互相传递文件或者文本信息，那该怎么做呢？我想刚刚的例子中，你会想到，只要建一个桥就行了。那两台电脑直接的桥怎么建？首先你是需要将两台电脑用线"连"起来。正常来说，有了这条传输的线，两台机器之间互相通信的基础条件就具备了。然后再进一步，你家里又买了几台电脑，这几台电脑互相之间也需要通信，那电脑只有一个插口，总不能与谁通信的时候，就插谁的网线吧，这要是都在同一时间通信，那不就的乱了？所以家里一般会有个路由设备，大家都在路由设备插入，然后要发的消息就先发个路由器，路由器再转发。那路由器怎么分清是哪台电脑？所以是不是大家要有个唯一标识符号（mac地址）。你看OSI七层模型就出来了。具体的化大家可以参考一下这个(https://www.lifewire.com/layers-of-the-osi-model-illustrated-818017)，我们主要是谈容器网络。

前面这些铺垫，其实主要还是为了主题容器网络。
<!--more-->
容器的本质是一种特殊的进程。容器里面最大的三个相关技术：限制、隔离、根文件，分别对应Cgroups、Namespace、RootFS。 而我们今天主题中，就需要先从 Network Namespace 讲起。

容器运行的时候的，它能看到的"网络栈" ，实际就是被隔离到它自己的 Network Namespace  中。网络栈应该包含有网卡（network interface)、回环设备（loopback device）、路由表（routing table）和 iptables规则。对于一个进程来说，这些要素就构成了它发起和响应网络请求的基本环境。

容器其实也可以直接使用宿主机的网络栈，只需要使用`-net=host` 就行。像这样容器直接使用宿主机网络栈的方式，可以为容器带来良好的网络性能，但是也不可避免的带来了共享网络资源的问题，比如端口冲突。所以我们希望的是，每个容器进程都使用自己的Network Namespace里面的网络栈，有自己独立的IP地址和端口。那这个时候就有一个情况出现，如果启动了两个容器，两个容器之间怎么互相通信呢？

如果你把容器看成一个主机，那就会想到，能否给两个容器插一根网线呢？让我们把视角切换到一台服务器上，你在服务器上安装了docker， 然后使用docker创建很多容器。你现在想让容器直接能够互相通信，那我们就需要有一个"交换机"，而容器是在服务器里面的，那我们能否也在服务器里面创建一个交换机了？在Linux设备中，起到虚拟交换机作用的网络设备，就是网桥（Bridge），Docker 项目会在宿主机上创建一个默认的 docker0 的网桥。那OK，交换机的问题解决了，那把容器和网桥进行连接的"线"呢？嗯，这个时候我们就要用到一个叫做Veth Pair 的虚拟设备了。Veth Pair 设备的特点是，它被创建出来后，总是以两张虚拟网卡(veth peer)的形式成对出现的。并且，从其中一个"网卡"的发出的数据包，可以直接出现在对应的另一张"网卡"上，哪怕这两个"网卡"在不同的Network Namespace 里面。

![图 5](/images/qiniu/1679408322281-%E5%8D%95%E4%B8%BB%E6%9C%BA%E5%AE%B9%E5%99%A8%E4%BA%92%E7%9B%B8%E8%AE%BF%E9%97%AE.png)  



OK，那我们现在就能让同一台主机上的不同容器直接互相通信了。那如果是不同主机上的不同容器直接怎么访问呢？比如K8S集群中，多台机器上，有多个容器，容器间的网络互相访问应该怎么弄？这就是我们常说的容器跨主机网络访问了。说到容器的跨主机通信，就有一个用的非常广的容器网络项目：Flannel。



Flannel提供了三种后端实现：1、VXLAN；2、Host-GW；3、UDP。

3种实现中，我们先看最早的一种方式：UDP。 

假设有两台主机：10.59.100.36、10.59.100.37。

Flannel 安装完之后，会在主机上创建一个Flannel0的设备，它是一个TUN设备(Tunnel 设备)。TUN设备是一个工作在三层的虚拟网络设备。它的功能非常简单，在操作系统内核和用户程序中传递IP包。 

当操作系统将一个IP包发送给flannel0设备后，flannel0就会把这个包转给创建它的flannel进程，这是一个从内核态到用户态的的流动方向；相反的如果flannel进程向flannel0发送一个ip包，那这个包就出现在了宿主机网络栈中，然后根据宿主机的路由表进行下一步处理，这是一个从用户态到内核态的流动方向。

那当flanneld收到flannel0转过来的IP包，然后怎么转发到这个IP了？这就是我们在flannel中的主要配置：子网的功劳了。主网与宿主机的关系保存在etcd中：`etcdctl ls /coreos.com/network/subnets` 所以我们可以看到，当a主机的flanneld收到ip包之后，只要将ip进行匹配就能知道要转发到那个主机上。这个流程能完成的原因就是每个主机上都启动了一个flanneld，都监听这8285的一个端口，，所以flanneld只要把udp包发往b主机的8285端口就行。而接下来，flanneld再把这个IP包发送给它管理的TUN设备，也就是flannel0设备，就可以找到对应的容器了。以下为示意图：

![图 2](/images/qiniu/1679408009109-flannel-%E8%B7%A8%E4%B8%BB%E6%9C%BA%E9%80%9A%E4%BF%A1-udp.png)  


但是为什么udp模式最后被放弃了？其实tun设备还是一个在三层的Overlay网络，比起两台宿主机的直接通信，flannel还做了一个通过flanneld进程进行封装的过程，这也导致flannel性能不好的直接原因，它需要在用户态和内核态直接进行3次转换。

![图 3](/images/qiniu/1679408053501-flanned-udp-%E5%BA%9F%E5%BC%83%E7%9A%84%E5%8E%9F%E5%9B%A0.png)  



所以flannel后续支持VXLAN，即：Virtual Extensible LAN（虚拟可扩展局域网），是Linux内核本身就支持的一种网络虚拟化技术。VXLAN的设计思想是在现有的三层网络之上，覆盖一层虚拟的、由内核VXLAN模块负责维护的二层网络，使得这个二层网络上的虚拟机或者容器，可以像局域网一样自由通信。而为了能够在二层网络上打通隧道，VXLAN 会在宿主机上设置一个特殊的网络设备作为"隧道"的两端，这个设备就叫做VTEP，即：VXLAN Tunnel End Point（虚拟隧道端点）。

VTEP设备其实有点类似之前说的TUN设备，不过它是作用在二层上，对二层的数据帧进行封装和解封，而且这个工作全部是在内核里完成的。

![图 4](/images/qiniu/1679408065526-flannel-vxlan%E6%A8%A1%E5%BC%8F.png)  

可以看到每个主机都有一个flannel.1的设备，就是vxlan所需的vtep设备，它既有IP地址，也有mac地址。

现在容器 A 要访问容器 B ，流程是怎样的呢？

首先 A 发出的IP 包，会先出现在docker0网桥，然后被路由到本机设备 flannel.1 进行处理，这个我们可以叫它原始包。

在到达 flannel.1之后，flannel.1 的作用就是要将这个原始包发送到正确的 目的宿主机的 vtep 设备上。

那怎么找到 vtep 设备呢？ 根据`route -n` 路由记录，我们可以知道目的主机的 vtep 设备的 IP 地址，而根据三层的 IP 地址查询对应的二层 MAC 地址信息，则恰好是ARP 表的功能。这里的 arp 记录，每个主机的 flannel.1 进程维护的，一台主机新加入到集群，就会把相应的 vtep 设备对应的 arp 记录下发到所有宿主机上。现在有了 MAC 地址，Linux 内核就可以开始二层封包操作，它会把容器的 IP和目的 VTEP 设备的 mac 地址组装成一个新的二层数据帧。封包过程只是增加二层头，不会改变原始包。

但是吧，这次的封包，其实对于宿主机的网络来说，并没有意义。Linux 内核还需要将这个二次封包的数据帧进行再一次封包成普通数据帧，这样才能让它把二次封包的数据帧，通过宿主机的 eth0 网卡进行传输。在内核进行封包成普通数据帧的时候，会将 VXLAN 的数据表进行标记，增加一个特殊的 vxlan 头，里面的重要标识 VNI ，它是 VTEP 设备识别某个数据帧是否应该归自己处理的标识。在 Flannel 中，VNI 的默认值就是 1， 这也就是为什么 flannel 的 vtep 设备叫 flannel.1的原因，这里的 1，就是 VNI 的值。不过在这里，flannel.1设备只知道另一个 flannel.1设备的mac 地址，却不知道对应的宿主机地址是啥。也就是说，这个 UDP的包应该发给哪台机器呢？在我们这种场景下，flannel.1 设备实际就扮演了一个网桥的设备的角色，在二层网络进行 udp 包的转发。而在Linux 内核里面，“网桥”设备进行转发的依据是根据FDB （Forwading Database）库来的。不难想象，这个 FDB 是由 flanneld 进程维护的:
```shell
# 在Node 1上，使用“目的VTEP设备”的MAC地址进行查询
bridge fdb show flannel.1 | grep 5e:f8:4f:00:e3:37
```
这样找到了目的宿主机的IP，然后宿主机再知道 arp 表要学习的内容知道 宿主机 ip 的 mac 地址， 之后就只需要进行正常 UDP 封包操作流程就行。接下来，node1 上的 flannel.1 设备就可以把这个数据帧从 node1 的 eth0 网卡上发出去，来到 node2 的 eth0 网卡。在 node2 的内核网络栈会发现这个数据帧里有 vxlan header，并且 VNI =1。所以 linux 内核会对它进行拆包，拿到内部二次封包后的数据帧，然后根据 VNI=1，将其交给 node2 的 flannel.1 设备进行处理。而 flannel.1 会进行进一步的拆包，取出原始数据包，流程跟之前一样，最后 原始数据包 进入到 container-2 的 network namespace 中。









