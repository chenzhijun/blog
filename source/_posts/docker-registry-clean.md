---
title: Docker 镜像仓库清理
copyright: true
date: 2019-06-30 16:07:52
tags: Docker
categories: Docker
---

# Docker 镜像仓库清理

公司使用registry作为镜像仓库管理，由于只对内使用并且只暴露给jenkins，不对其它人公开，因此也就没有做registry的限制。不过由于我们在jenkins构建的时候使用docker 的一个插件，但是我们公司又系统在构建的时候不还tag，因此造成同一个名字有很多历史的layer都保存在了镜像仓库中，占用的空间随着时间越来越大。经过这次清理，由原来的占用1.9T清理空间到占用195G，效果还是非常的明显。

清理的步骤如下，下面的步骤适用于镜像名和tag都相同，然后重复push的情景下：
<!--more-->
## 清理多余的manifest

适用命令：
`docker run -d -v /data/registry:/registry -e REGISTRY_URL=http://{{registry-ip}}:5000 mortensrasmussen/docker-registry-manifest-cleanup:latest`

`mortensrasmussen/docker-registry-manifest-cleanup:latest`是一个开源的工具：[docker-registry-manifest-cleanup](https://github.com/mortensteenrasmussen/docker-registry-manifest-cleanup)。其中`-v /data/registry:/registry` /data/registry 是镜像仓库registry使用的存储在主机上的目录。

> ps:你的registry 是用容器的方式跑的：`docker run -v /data/registry:/registry -p 5000:5000 xxxxx/registry:latest` 这个就是`/data/registry`在本地存储的位置。

## registry 的清理

执行完上面的步骤还不够，还需要调用registry的清理功能才能实际释放空间:`docker exec registry /bin/registry garbage-collect /etc/docker/registry/config.yml`

`config.yaml`:

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
  delete:
    enabled: true
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

执行完之后再使用`df -h`你就可以看到磁盘的空间被释放出来了。