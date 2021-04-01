---
title: 随笔-8-19
date: 2017-08-19 19:02:11
tags:
	- Question
categories: Question
---

## 8月19日 阶段总结

### 日期格式?

SimpleDateFormat ===> SimpleDateFormat


### Double 后面有个E?

A: double 为啥后面有个E ,E后面的数字代表后面乘以10的位数

### java 格式化金额显示

A: numberFormat,留4位小数:

```

    public static final int DEFAULT_FORMAT = 4;

    public static String formatCurrency(Double money)
    {
        NumberFormat numberFormat = NumberFormat.getNumberInstance();
        numberFormat.setMaximumFractionDigits(DEFAULT_FORMAT);
        return numberFormat.format(money);
    }

```

### 查看服务器上java命令位置:

A: `whereis javac/java`    `ls -al` 一直找到没有软连接位置

### jenkins 构建的时候,nohup: failed to run command 没有找到目录,没有权限

A: jenkins启动springboot脚本

```
    #!/bin/bash
    source /etc/profile
    #pname=$1
    pname=com.apec.warehouse-server-1.0-RELEASE.jar
    puser=root
    pid=`ps aux | grep $pname | grep $puser | grep -v grep | awk '{print $2}'`

    if [[ -z $pid ]]; then
        echo "I can NOT find $pname running by $puser"
    fi
    #/home/cncsen/warehouse/com.
    chmod 777 /home/cncsen/warehouse/com.apec-warehouse-server-1.0-RELEASE.jar
    kill -9 $pid >/dev/null 2>&1
    cd /home/cncsen/warehouse/
    exec nohup java -Xmx128m -Xss256k -jar $pname --cacheType=single  --spring.profiles.active=test 5 >>server.log &
    tail -f server.log
```



