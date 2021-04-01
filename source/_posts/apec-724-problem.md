---
title: 问题总结
date: 2017-07-24 14:43:42
tags:
	- Question
categories: Question
---

## Java问题总结

### 7-14
 1) idea编码不一致

 A: FileEncoding -->globalEncode&&projectEncoding-->Default encoding for properties UTF-8
 _**所有项目尽量一开始就设置UTF-8再导入项目**_

2) idea vim插件冲突的问题，

A:直接设置vim emulation。 将handler全部改为ide

3) redis.clients.jedis.exceptions.JedisConnectionException: no reachable node in cluster

A: 没有找到可以用的redis集群

4) idea 查看当前类的信息

A: `alt+Q` 可以快速查看当前是哪个类

5) idea ctrl+o 重写某个方法

### 7-15
 1) spring定时任务， CRON表达式  `*/5 * * * * ?`

 A:`CRON`的格式是固定的，如果需要在spring中开启定时任务，给类加上`@Service`注入，然后在方法上加入`@Scheduled(cron = "*/5 * * * * ?")`就可以了开启一个定时任务了。

 2)  java list排序自定义

 A:如果有需求需要自己给list里面的对象设置排序。可以重写方法。

 ```
 Collections.sort(pushData, (AppTips o1, AppTips o2) ->
            {
                if(o1.getCreateDate().before(o2.getCreateDate()))
                {
                    return 1;
                }
                return 0;
            });
 ```

 3) java传入引用，没有修改值,传入一个list到方法里面，在方法里面修改了list，之后list的值没有改变，引起误解为啥传入对象，修改了对象的值而没有改变对象内容。

```
import java.util.ArrayList;
import java.util.List;

public class ListTest
{
    public static void main(String[] args)
    {
        List<String> a = new ArrayList<String>();
        a.add("adf");
        a.add("werwerwe");

        List<String> b = new ArrayList<>();
        setB(b);
        System.out.println(b.isEmpty()); // true

        setB2(a,b);
        System.out.println(b.isEmpty()); // false


    }


    private static void setB(List<String> b)
    {
        b=new ArrayList<>();
        b.add("test1");
        b.add("tet2");
    }

    private static void setB2(List<String> a,List<String> b){
        b.addAll(a);
    }
}

```

 A: 其实Java除基本类型之外，其他的都为引用传递。这是对的，在上面的例子中，因为在`setB`中，我们将引用b重新指定到了一个对象`new ArrayLsit<>()`所以b此时已经不再指向原来的空间了。

 4) redis有一个database

 A: 一开始直接用`redis-cli`,用`keys *`发现没有值可以获取，但是代码里面可以获取到值，后来发现其实springboot做了设置`redis.database=4`。其实在redis客户端里面使用`select 4`就可以了。

 ### 7-21

 1)requestParam 参数可以放在body里面

 2)nested exception is java.lang.IllegalArgumentException: Not enough variable values available to expand

 A: 这个异常耽误了我半天的时间，后来发现其实是restTemplate使用的时候url不要拼接json串，因为它遇到`{}`会认为是一个变量，然后去解析值。[详情](https://w3stacks.com/questions/spring/70373/using-resttemplate-in-spring-exception-not-enough-variables-available-to-expand)
