# 注解分类

1. jdk自带注解：@Override, @Deprecated, @.....
2. 第三方注解： spring,mybatis.
3. 自定义注解：

# 注解的定义

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited // 继承使用
@Documented // 针对文档的
@Rep
public @interface xxx{

}
```

# 注解的属性

属性只能是基本类型，String，enum，不能赋值为null

全部定义为方法，必须赋默认值，或者在声明的时候赋默认值；

# 标记注解

junit

##实例


# 方法

## 检查参数的有效性


## 方法签名


## 慎用重载


## 返回0长度数组或集合



## 所有的api文档写注释