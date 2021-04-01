---
title: 基础脚本
copyright: true
date: 2018-05-10 21:16:52
tags: shell
categories: Linux
---

# 基础脚本

最近在同事那里学到一个小脚本，感觉要是之前我也会这样写，那我省去多少时间啊，技多不压身啊。

<!--more-->

```shell

#!/bin/bash
echo "ok"

buildProcess(){
	echo "build image";
	docker build -t promethues-agent:v0.1.test .
	echo "start contianer..";
	docker run -dit -P --name pro-agent-test promethues-agent:v0.1.test --spring.profiles.active=test
	echo "finished";

}

privilege(){
	if [ "$(whoami)" != "root" ]
	then
        echo "should be root to execute this script";
		exit 1
	fi
}

stopProecess(){

    echo "delete container..";
    docker rm -f pro-agent-test
    echo "delete image...";
    docker rmi promethues-agent:v0.1.test
    echo "finished";
}

showUsage(){
	cat <<END
Usage: $0 <build|stop>

END
}

main(){
	command=$1
	
	# check privilege
#	privilege

	case $command in
    		"build") buildProcess;;
    		"stop") stopProecess;;
    		*) showUsage;;
	esac
}
main $@

```

上面是一个docker镜像的制作脚本，命名为`dockerimg.sh`。使用方式：

```shell

./dockerimg.sh build

./dockerimg.sh stop

./dockerimg.sh *

```