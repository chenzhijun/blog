---
title: 从命令行参数获取Shell脚本参数
date: 2018-01-08 17:03:58
tags:
    - Shell
categories: Linux
---

# 从命令行参数获取Shell脚本参数

很多时候我们写完脚本都需要指定一些变量，这些变量可能根据实际的环境而值不同。所以如何在 shell 脚本中接收到不同的变量值了？

## 获取参数

shell 脚本获取参数的方式很多，我只记录我用过的一种：`filebeat.sh`

```shell
#!/bin/bash
if [ ! -n "$1" ] ;then
    echo "请输入日志文件目录,比如/path/to/xxx.log,输入/path/to:"  
    exit 1;
fi

echo "日志文件目录：$1" #log_path

if [ ! -n "$2" ] ;then
    echo "请输入项目名称,比如customer,notice ...:"  
    exit 1;
fi

echo "项目名称:$2" #log_type

cd $(pwd)/filebeat
# copy a filebeat.yml to filebeat_log_type.yml
cp config/filebeat.yml config/filebeat_$2.yml
sed -i "s#PATH#$1#g" config/filebeat_$2.yml
sed -i "s#DOCTYPE#$2#g" config/filebeat_$2.yml
nohup ./filebeat -e -c config/filebeat_$2.yml >> logs/$2.log &
tail -f logs/$2.log
```

shell 文件都是用`#!/bin/bash`作为第一行，我们在 filebeat.sh 中获取两个参数，如何使用这个脚本了？
`sh filebeat.sh param1 param2`

param1 我们在脚本中使用`$1`获取，param2 我们在脚本中使用`$2`获取，以此类推。

如果要在文件中替换某个特殊文字或者字符如果值中带有`/path/to/file`,这种带有`/`的字符，在脚本中可以使用`#`来转义。之前用的是`sed -i "s/path/$1/g" config/file.yml`,后来改成`sed -i "s#path#$1#g config/file.yml`。


参考文档：
[http://www.jb51.net/article/56549.htm](http://www.jb51.net/article/56549.htm)

[https://www.jianshu.com/p/d3cd36c97abc](https://www.jianshu.com/p/d3cd36c97abc)

[http://blog.51cto.com/w55554/1223870](http://blog.51cto.com/w55554/1223870)
