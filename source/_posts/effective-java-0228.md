---
title: Effective Java 0228
date: 2018-02-28 14:30:33
tags:
    - Java
categories: Java
---

# Effective Java 0228

## 覆盖equals时候请遵守通用规定

    翻开书，看到了书的第3章-对于所有对象都通用的方法。我们知道，java中一切类都继承自Object，在Object类里面有
    两个非常重要的方法：`hashCode()`,`equals()`;

    其实Object类里面很多方法的注释非常详细，有一些比较通用的约定。作为我们在重写的时候我们最好是遵守这些约定。

如果我们要覆盖equals方法，我们需要思考几个问题：
1. 我们为什么要覆盖？可不可以不覆盖？各自场景是什么？
2. 怎么覆盖？
3. 有什么需要特别注意的？

覆盖equals看起来非常简单，其实坑有蛮多。最好的避免方式当然就是不覆盖了，这样我们可以看到在Object里面的实现：

```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

默认equals比较的就是类只与它自身的相等。在何种情况下我们不需要覆盖呢？书上说有四点：
1. 类的每个实例本质上是唯一的。
2. 不关心类是否提供“逻辑相等”的测试功能。
3. 超类已经覆盖了equals，从超类继承过来的行为对于子类也合适。
4. 类是私有的或是包级私有的，可以确定它的equals方法永远不会被调用。


什么时候需要覆盖，那么肯定就是上面四个的反例或者其它的情况了。一般我们需要比较逻辑上的关系时候，我们可能需要重写equals。这种被称为“值类”。值类也有不需要覆盖的场景:比如单例模式，每个值至多存在一个对象。那么比较值相等的意义就不大了;另一种就是枚举，枚举类型逻辑相同与对象等同是一个意义，因此这两个方式就算不覆盖Object的equals方法也可以。

覆盖equals时候最好也是必须遵守它的通用约定：
1. 自反性，x.equals(x) 返回true
2. 对称性，x.eq(y),y.eq(x) 返回true
3. 传递性，x.eq(y),y.eq(z),x.eq(z) 返回true
4. 一致性，多次调用x.eq(y),都应该返回true
5. 与null进行equals(null)的时候必须返回false

高质量equals方法的建议：
1. 使用==操作符号，查看是否当前比较参数是本身这个对象的引用。如果是返回true。
2. 使用instanceof操作符检查“参数是否是正确的类型”，这个可以帮助我们排序非当前类型的比较，也可以排除null值。
3. 转换instanceof之后的类型为当前this指向的对象的类型。
4. 检查参数中的每个域，是否和该对象中对应的域相等。

对于不是float和double的基本类型可以用“==”比较，另两个调用他们的compare方法(为什么compare方法可以了？)。
另外可以将最有可能不一样的域提前比较。

覆盖equals的时候一定要一定要一定要覆盖**hashCode()**。

那么hashCode()该怎么覆盖了？下次分享。