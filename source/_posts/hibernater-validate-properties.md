---
title: Hibernate 校验参数
date: 2018-01-31 19:40:27
tags:
    - Validator
categories: Java
---

# Hibernater Validator 校验参数


## 使用方式

如果我们使用spring mvc 那么肯定知道在方法中我们可以使用注解对参数进行校验：

```java
@PostMapping("/user)
public void addUser(@Valid User user){
    User userP = user;
    return ;
}
```

这个时候通常是在User对象里面使用一些注解来判断如：@NotNull,@NotEmpty。这些bean验证方法是遵循JSR303和JSR380规范的，目前的情况可以去这里查看详情：[http://beanvalidation.org/2.0/](http://beanvalidation.org/2.0/)。

实现规范的这些中有一个包是[Hibernate Validator](http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-customconstraints-simple)，这个包不是我们的ORM框架，可能是orm太出名了，以致于提到Hibernate大家都会想到SSH的Hibernate ORM，其实Hibernate Validator是一个非常完善的bean Validator。如果我们不想自己去实现一套，其实是可以引入这个包的。开源社区遇到的情况肯定会比我们自己要多。而且它也支持我们自定义注解来进行校验。

在我们的系统中，是无法采用@Valid这种方式的，因为我们有一些特别的操作处理所以不能使用它。如果你想自己使用的话，可能可以像我这样，写一个Validtor工具类，然后在需要的地方调用：

```java
public class ValidatorUtil {
    private static final Validator VALIDATOR = Validation.byProvider(HibernateValidator.class).configure().failFast(true).buildValidatorFactory().getValidator();

    public static <T> void validate(T obj) {
        Set<ConstraintViolation<T>> validateResult = VALIDATOR.validate(obj);
        if(!validateResult.isEmpty()){
            System.out.println("message:"+validateResult.iterator().next().getMessage());
            System.err.println("messageKey:"+validateResult.iterator().next().getMessageTemplate());
            throw new RuntimeException(String.format("参数校验失败:%s", validateResult.iterator().next().getPropertyPath().toString()+validateResult.iterator().next().getMessage()));
        }
    }
}
```

之后你可以在属性上面这样定义：

![2018-01-31-20-00-07](/images/qiniu/2018-01-31-20-00-07.png)

如果需要使用校验的，只需要使用<!--more-->

```java
ValidatorUtil.validate(obj);
```

如果有异常它会直接抛出来。

## 自定义注解

如果在某些场景下，提供给我们的注解不够用，那我们是否可以自己进行扩展了？ 好的架构，是允许做扩展的。validator也可以支持自定义注解。自定义注解详解在这里[自定义验证规则注解](http://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-customconstraints)。总的来说就是三步：

1. 创建一个注解，并且定义一些校验规则；
2. 继承Validator；
3. 定义默认的error message

现在我们尝试定义一个自定义注解：

0. 准备，定义一个枚举，接下来用到：

```java
package com.chenzhijiun.validator.customer;

/**
 * @author chen
 * @version V1.0
 * @date 2018/1/30
 */
public enum CaseMode {
    UPPER,
    LOWER;
}
```

1. 创建一个注解:`CheckCase`：

```java
package com.chenzhijiun.validator.customer;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Target({ FIELD, METHOD, PARAMETER, ANNOTATION_TYPE, TYPE_USE })
@Retention(RUNTIME)
@Constraint(validatedBy = CheckCaseValidator.class)
@Documented
public @interface CheckCase {

    String message() default "{org.hibernate.validator.referenceguide.chapter06.CheckCase." +
            "message}";

    Class<?>[] groups() default { };

    Class<? extends Payload>[] payload() default { };

    CaseMode value();

}
```

2. 继承Validator,并且将实现自己的规则逻辑

```java
package com.chenzhijiun.validator.customer;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

/**
 * @author chen
 * @version V1.0
 * @date 2018/1/30
 */
public class CheckCaseValidator implements ConstraintValidator<CheckCase, String> {

    private CaseMode caseMode;

    @Override
    public void initialize(CheckCase constraintAnnotation) {
        this.caseMode = constraintAnnotation.value();
    }

    @Override
    public boolean isValid(String object, ConstraintValidatorContext constraintContext) {
        if ( object == null ) {
            return true;
        }

        if ( caseMode == CaseMode.UPPER ) {
            return object.equals( object.toUpperCase() );
        }
        else {
            return object.equals( object.toLowerCase() );
        }
    }
}
```

可以看到，我们这里的主要逻辑就在isValid中处理的。

3. 定义error message ，这个错误提示在哪里定义了？我们可以看到在`org.hibernate.validator.resource`下有很多的properties文件，
这是因为Hibernate做了国际化处理。
![2018-01-31-20-14-03](/images/qiniu/2018-01-31-20-14-03.png)

如果我们需要重写，只需要将其中的中文的属性文件进行重写放到resource目录就可以了。在定义的注解中我们可以将message指定为我们复制的properties文中
的key，这样就能完成error message的自定义了。

源码：https://gitee.com/chenzhijun/validator