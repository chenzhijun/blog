---
title: Docker Registry 磁盘空间清理
copyright: true
date: 2019-05-09 23:06:39
tags: Docker
categories: Docker
---

# Docker Registry 磁盘空间清理

使用 Docker 的时候用的是 Docker Registry 来存储镜像。一开始的时候给了500G数据盘，日积月累的累积的数据就开始变多了。
没办法只好想办法去清理。看了下官网的api文档：[Docker Registry API](https://docs.docker.com/registry/spec/api/)

其实还挺简单的，主要是用http接口先将镜像和层删除，然后使用镜像仓库的garbage-collect。

默认HTTP接口是不支持DELETE方法的，需要修改配置文件中`storage.delete.enabled: true`，配置文件的解释可以在这里看到地址[Registry 配置文件](https://docs.docker.com/registry/configuration/#delete)

```yaml
storage:
  delete:
    enabled: true
```

镜像仓库的[garbage-collection](https://docs.docker.com/registry/garbage-collection/)可以看下官网文档。主要看懂一个图就可以了：

```
A -----> a <----- B
    \--> b     |
         c <--/
```

```
A -----> a     B
    \--> b
         c
```

这样c就要回收了。

然后我们使用HTTP的API。查到repo，tags，然后就可以删除相应的镜像。

但是在删除的时候要注意下，首先我们要通过接口获取digest的值，但是这个digest的值获取方式比较特别，首先我们访问：`/v2/<name>/manifests/<reference>`这个接口的时候，需要使用GET获取HEAD方法，然而在请求的时候需要加入Header：`Accept: application/vnd.docker.distribution.manifest.v2+json`这样才可以在返回的HEAD中才能获取到正式的digest。

使用golang写了一个demo代码：

```golang

package main

import (
"encoding/json"
"flag"
"fmt"
"io/ioutil"
"net/http"
"strings"
)

type Tag struct {
	Name string   `json:"name"`
	Tags []string `json:"tags"`
}

type Repo struct {
	Repositories []string `json:"repositories"`
}

//registry garbage-collect /etc/docker/registry/config.yml
//registry garbage-collect --dry-run /etc/docker/registry/config.yml > test.log
//cat test.log | awk -F : '{print $1}' | sort | uniq -c | sort -rn -k1 | head -10
func main() {
	registryUrl := flag.String("url", "http://registry.xxxxx.com:5000", "registry url")
	pattern := flag.String("pattern", "", "删除镜像名中有pattern的镜像")
	flag.Parse()
	fmt.Println("registry url:", *registryUrl, ",pattern:", *pattern)
	resp, _ := http.Get(*registryUrl + "/v2/_catalog?n=10000")
	bytes, _ := ioutil.ReadAll(resp.Body)
	r := Repo{}
	json.Unmarshal(bytes, &r)
	for _, v := range r.Repositories {
		if *pattern == "" || len(*pattern) <= 0 {
			*pattern = "2017"
		}
		fmt.Println("pattern:", *pattern)
		if strings.Contains(v, *pattern) {
			url := fmt.Sprintf(*registryUrl+"/v2/%s/tags/list", v)
			fmt.Println("url" + url)
			resp, _ := http.Get(url)
			bytes, _ := ioutil.ReadAll(resp.Body)

			var t = Tag{}
			fmt.Println("byteStr:", string(bytes))
			json.Unmarshal(bytes, &t)
			fmt.Println("tag:", t)
			client := http.DefaultClient
			if len(t.Tags) > 0 {
				for _, ti := range t.Tags {
					mainfests := *registryUrl + "/v2/%s/manifests/%s"
					url = fmt.Sprintf(mainfests, t.Name, ti)
					request, _ := http.NewRequest(http.MethodGet, url, nil)
					request.Header.Set("Accept", "application/vnd.docker.distribution.manifest.v2+json")
					response, _ := client.Do(request)
					digest := response.Header.Get("docker-content-digest")
					fmt.Println("digest:", digest)
					fmt.Println("headers:", response.Header)
					deleteUrl := fmt.Sprintf(mainfests, t.Name, digest)

					fmt.Println("deleteUrl:", deleteUrl)
					request, _ = http.NewRequest(http.MethodDelete, deleteUrl, nil)
					response, _ = client.Do(request)
					headers := response.Header
					fmt.Println(response.Status, headers)

				}
			}
		}

	}

}


```

就算这样执行完，别忘记了，进入到registry的容器中，然后使用：
```shell

registry garbage-collect /etc/docker/registry/config.yml

```

`/etc/docker/registry/config.yml`文件要打开之前说的`storage.delete.enabled: true`
