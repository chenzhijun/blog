---
title: 使用jenkins对springboot进行CI
date: 2017-08-10 20:40:49
tags:
    - jenkins
---


## 写给小白看的jenkins CI教程

### 目标

使用jenkins持续部署一个springboot项目,项目使用jar包发布.从安装jenkins的机器将jar包发布到指定的服务器目录,并且使用脚本运行springboot. (_前面几张图可能看不见,以后补上_)

### 个人背景

Java工程师毕业一年,Linux一般熟悉,Java初阶,所以不用担心难度,如果有疑问可以联系我:`vbookchen@gmail.com`

### 系统环境与准备

__jenkins就是相当于一个web服务,与环境没有太大关系__

1) Linux/Windows 操作

2) jenkins.jar/jenkins.msi //安装就是了,其他的是系统可以下载war包在tomcat里面运行

3) gitlab.com 账号

4) gitlab 公有仓库放置 spring-boot 源码

5) maven // 可以用jenkins安装也可以用本地的

6) JDK 安装jenkins的主机必须要有jdk, docker版本除外

7) 部署服务的Linux服务器

<!--more--> 

### 基础搭建

安装好jenkins之后,windows是会在服务里面可以看到一个jenkins服务的.占用端口8080.Linux的没操作过,docker版本的jenkins镜像记得映射本地端口. mac版本应该也是在8080端口,可以看到服务.

登录`localhost:8080`

看到一个初始化页面,页面会提示初始化密码的问题,一串很长的字符:

![初始密码](/images/jenkins/img1.png)

输入密码之后会让我们选择安装模式,安装插件,我们选择默认模式:

![插件选择](/images/jenkins/img2.png)

稍等一会,根据网速快慢不一样,等待时间不一样,之后选择创建用户密码:

![创建用户](/images/jenkins/img3.png)

__ps:这里记得选择右边`Save And Finish`那个,左边的`continue as admin` 这个时候是没有创建的,默认账号密码为admin/初始密码__

安装成功后可以说第一步完成.

### 插件安装

默认安装的插件是没有maven和ssh插件的,所以需要重新安装:

1) 安装maven插件,maven integration plugin:

![安装maven插件](/images/jenkins/img4.png)

2) 安装ssh插件,publish over ssh:

![安装ssh插件](/images/jenkins/img5.png)

3) 安装git插件,默认的插件安装列表已经安装了

4) 如果插件安装不成功(gfw,国内有时候是这样),可以到`系统管理--插件管理--高级--上传插件`;插件下载地址:[jenkins插件下载地址](https://updates.jenkins-ci.org/download/plugins),找到对应的插件hpi文件下载后上传就可以了.


安装之后第二步完成

### 环境配置

配置好基本环境可以省很多时间,看到jenkin主页的左侧有很多选项:

1) 配置全局属性,`系统管理-->系统设置`看到`Environment variables`,加入LANG=zh_CN.UTF-8,其实可以先去`系统管理-->系统信息`找到file.encoding,看看属性是不是utf-8.

![全局属性](/images/jenkins/img6.png)

2) 配置ssh信息,`系统管理-->Publish over ssh` 如果没有`publish over ssh`那么是第二步中插件没有安装成功.

![ssh信息](/images/jenkins/img7.png)

3) 配置JDK信息,`系统管理-->Global Tool Configuration`,选中`jdk--jdk安装`

![jdk安装](/images/jenkins/img8.png)

4) 配置maven项目,`系统管理-->Global Tool Configuration`,选中`maven--maven安装`

![maven安装](/images/jenkins/img9.png)


### 新建任务

基础打好后,开始建楼

1) jenkins主页左边栏:`新建`建立第一个CI任务.

![新建任务](/images/jenkins/img10.png)

2) 对任务项做相关配置:

```
    a) 任务名称/介绍;
    b) 源码地址(git/svn);
    c) 构建触发器;
    d) Build;
    e) PostSteps; 配置需要将生成的jar包等上传到哪个服务器,执行哪个脚本 ;
    f) 保存

```

![任务项目配置](/images/jenkins/img11.gif)

__更正下配置,maven 构建那里跳过测试,应该是 -Dmaven.test.skip=true  -X__

3) 成功后的页面:

![成功的页面](/images/jenkins/img12.png)

4) 点击`立即构建---build history---console output`可以看到构建的输出

_ps 附录一段脚本_

stop.sh:

```
#stop.sh
#!/bin/bash
echo "Stopping SpringBoot Application"
pid=`ps -ef | grep girl-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}'`
if [ -n "$pid" ]
then
   kill -9 $pid
fi

```

start.sh:

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.141-1.b16.el7_3.x86_64/jre
echo ${JAVA_HOME}
echo "start startup girl"
chmod 777 /home/girl/girl-0.0.1-SNAPSHOT.jar
echo "grant right...."
cd /home/girl/
nohup ${JAVA_HOME}/bin/java -jar girl-0.0.1-SNAPSHOT.jar > /home/girl/server.log &

whatToFind=seconds
function watch(){
 
    tail -1f server.log |

        while IFS= read line
            do
                echo "buffering:" "$line"

                if [[ "$line" == *"$whatToFind"* ]]; then
                    echo $msgAppStarted
                    pkill tail
                fi
        done
}
watch

echo "start success"

```

问题：jenkin tail -f server.log的时候一直build unstable。 但是又想让控制台输出日志，所以在脚本里面加上一个watch方法，这样就可以查看到日志输出了。然后在jenkin job配置中添加![2017-09-08-17-10-37-201798171037](/images/qiniu/2017-09-08-17-10-37.png)

在执行脚本的时候添加`BUILD_ID=dontKillMe`;就在kill的时候保证jenkins主进程不被杀掉。


 如果是centos服务器,用的是`yum install java`安装的Java环境,查找`java_home`的方法:用`ls -al`一直找到软连接的真实指定位置:

![查找java_home](/images/jenkins/img14.png)


__记得要给路径或者脚本赋权限,不然jenkins可能出现__`nohup: failed to run command java: No such file or directory`,__这是没有权限造成的.__

