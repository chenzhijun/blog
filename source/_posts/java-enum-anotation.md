---
title: Java 枚举和注解总结
date: 2018-03-22 22:38:32
tags: Java
categories: Java
copyright: true
---

# Java 枚举和注解总结

## 枚举

### 没有枚举前我们基本上常量来定义值：

```java
public interface Color{
    public static final int GREEN = 0;
    public static final int RED   = 1;
    ....
    ....
    ....
} 
```

如果有了枚举后我们会怎样了？

```java
public enum Color{
    GREEN,RED,BLACk,.......
}
```

代码是不是就清晰很多了？而且使用的时候输出的值我们是可以使用GREEN，是不是可以很明白的知道是什么颜色？

### 枚举的创建

枚举是继承自`Enum`，使用`enum`关键字。

### 枚举的使用场景

枚举适合在**固定的常量**下使用，比如四季，月份，星期；这种基本公认的而且不会有改变的场景下使用。

### 编译器中的枚举

编译器默认帮我们实现了很多枚举中的方法，比如equals(),hashCode(),toString,values(),valueOf(String)等。这些都是编译器帮我们做的。

### 枚举中的注意事项

1. 不能使用static，final修饰枚举，因为它是隐式的final类型的；
2. 因为是final类型，所以我们也就知道它是**不能被继承**的；
3. 从Enum继承的clone是final类型的，枚举是不能重写clone方法的，并且Enum里面的clone方法直接抛出异常，所以enum是不能被clone的;
4. enum中的ordinal是强依赖于枚举实例的定义顺序的，所以用ordinal来做判断顺序是不推荐的，因为只要在非最后加入实例，那么就会改变整体的顺序;如果是需要顺序可以自定义属性。

<!--more-->

## 注解

### 注解出现前

注解的作用我理解为就是用来约定一些数据定义，让我们可以在某个属性或者某个地方做个标记。在注解出现前能做这个的应该是XML，我们通常在xml中定义类或者属性的相关配置。而有了注解我们可以在代码中直接定义了。

### 注解的分类

1. 定义注解的注解，元注解：`@Rentation`，`@Target`，`@Document`，`@Inherited`
2. jdk内置注解：`@Override`，`@Deprecated`...
3. 自定义的注解
4. spring等外部注解

作用场景：

`@Rentation`：Source，Class，Runtime

作用目标域：

`@Target`：Construct，Field，Local_variable,method,package,paramter,type

### 注解的定义

注解的定义使用`@interface`关键字，并且使用元注解进行标注：

```java
@Rentation(Rentation.Runtime)
@Target(ElementType.Field)
public @interface XxAnotation{
    String values() default "";
}
```

### 注解属性

注解里面的属性只能使用以下6种类型来定义：

```java
1. 所有的基本类型；
2. String
3. Class
4. enum
5. Annotation
6. 以上类型的数组类型
```

注解不允许使用基本类型的包装类来定义里面的注解属性。注解里面的属性都是使用方法的方式来定义的。有点类似接口方法。
注解的属性需要注意一下几点：

1. 要么具有默认值，要么在使用注解的时候提供属性的值；
2. 非基本类型的元素，默认值不能为null；
3. 如果只有一个属性，那么可以设置为value，在使用注解的时候就可以直接赋值；`@XxAnotation("ok")`

> ps:注解是不能继承的。

### 注解处理器

我们定义了注解，设置了元素值。那么就必须要有一个处理器来进行注解处理。这个可以看看之前的[Java 特殊字段脱敏](http://www.chenzhijun.me/2017/08/19/java-sensitive/)，我们可以使用反射来获取注解定义的值，然后进行业务处理。


























