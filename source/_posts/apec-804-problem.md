---
title: apec-804-problem
date: 2017-08-04 11:29:40
tags:
	- Question
categories: Question


---

## 7月25日问题

### Persistable QModel生成。?

A: 其实这是一个QSL框架生成的，所有Entity都会生成一个QEntity类。

## 8月1日问题

###  maven complier install package 的区别？

A:complier只是编译，不会生成jar包，package生成jar包，但是不会将包发布到仓库，只在项目下，install会将jar包发布到仓库让别人也可以用

### maven install 跳过测试?

A: 执行的时候加上 -Dmaven.test.skip=true

###  jpa 插入枚举值得时候默认为1？

A:默认使用 ordinal 值，其实应该为string

### maven 设置java版本？

A: pom文件中加入：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>

```

### springboot 使用jrebel热加载？

A: 现在pom文件里面加入`devtools`: 之后使用`ctrl+alt+shift+/`  选中`Registry`   勾上`app.running`

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>

```

## 8月3日问题

### SimpleDateFormate 日期格式类使用注意？

A:日期工具类里面使用了SimpleDataFormate=> format方法里面有个`calendar.setTime(date);`.  
如果是多线程中，这个方法没有做同步Thread-A，有一个date，然后Thread-B有一个date，那么造成出现预料之外的结果，当然这种情况出现在共享一个SimpleDateFormate当中，既设置一个全局的simpleDateformat。如果在方法中设置局部变量的simpledateformate，那么局部变量是线程安全的。
当然追求效率可以使用ThreadLocal,设置局部缓存

### Java 反射获取属性？

A:注解是分为执行状态（运行，编译），执行位置（类，方法，属性），如果用`declared`， 包括所有非父类属性。如果不用`declared`，获取所有为public的属性
`getFields()`===> 获取所有属性为public的  
`getDeclaredFields()`===> 获取所有非继承的数据(public,protect,default....)
