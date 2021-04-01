---
title: 重温Java编程思想的一些感悟-20180305
date: 2018-03-05 23:43:32
tags:
    - Java
categories: Java
---

# 重温Java编程思想的一些感悟-20180305

JIT(just-in-time) 即时编译技术：将程序全部或者部分翻译成本地机器码，程序运行速度因此得到提升。
对于hotspot来说，代码每次被执行，都会做一些优化，执行的次数越多，速度也就越快。

java会给所有的默认域做一个初始化，非基本类型对象不初始化值的情况下，默认为null。
初始化顺序:static->{}->构造器；

static的数据指向同一份存储区域，不能用于局部变量。

final对于基本类型，其数值永不改变。final对于引用类型，其引用永不改变。
必须在定义final或者在构造器中对final进行赋值。

final类无法继承，方法无法重写或修改。private的方法其实是隐式的final。

compareTo,如果是两个int，最好不要直接返回i-i2，如果是有符号的int类型，一个正数最大，一个负数最大，那么将永远返回负数。所以如果重写compareTo，最好是不要直接返回return i-i2;

集合是fail-fast的，比如：

```java
    list.add(aa);
    list.iterator();
    list.add(cc);
```

需要明确在添加或者修改完容器所有元素之后再获取迭代器。