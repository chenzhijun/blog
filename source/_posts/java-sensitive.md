---
title: Java 特殊字段脱敏
date: 2017-08-19 20:02:16
tags:
	- Java
categories: Java
---

## 利用Java给敏感字段脱敏

### 需求

客户信息中,有敏感字段比如身份证,银行卡,手机号码等字段要进行脱敏处理(加星号).

### 解决方案
<!--more-->
想实现一个工具类调用`XxxUtils.maskObject(obj)`;就可以实现脱敏,过程中想到的是用反射,然后给需要脱敏的字段加上注解.可能有需求要将星号变成美元符号,手机号码保留前3位与后四位,身份证保留前后,各一位,所以考虑在注解`@Sensitive`中加入3属性`sensitiveChar`替换字符,`prefixLength`前缀预留长度,`suffixLength`后缀预留长度;之后再`maskObject()`方法中进行反射之后将值进行脱敏处理.

定义的`Sensitive`注解:

```

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Sensitive
{
    /**
     * 是否打码,默认为true
     * @return
     */
    boolean mask() default true;

    /**
     * 前缀预留位数
     *
     * 比如:
     *          prefixLength=3
     *
     *          private String idCard="41234560987123543"
     *
     *          打码后为: idCard= 412**************
     *
     * @return
     */
    int prefixLength() default 0;

    /**
     * 后缀预留位数
     *
     * 比如:
     *          suffixLength=3
     *
     *          private String idCard="41234560987123543"
     *
     *          打码后为: idCard= **************543
     * @return
     */
    int suffixLength() default 0;

    /**
     * 敏感字符替换字符
     *
     * 比如:
     *          sensitiveChar="$",(suffixLength=3);
     *
     *          private String idCard="41234560987123543"
     *
     *          打码后为: idCard= $$$$$$$$$$$$$$543
     * @return
     */
    String sensitiveChar() default "*";
}

```

`Sensitive`主要为类的属性进行描叙,方便我们反射的时候获取预留信息.

定义的反射工具类`MaskUtils`:

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.StringUtils;

import java.lang.reflect.Field;
import java.util.ArrayList;
import java.util.List;

/**
 *
 * @author chen
 * @version V1.0
 * @date 2017/8/17
 */
public final class MaskUtils
{
    private static final Logger logger = LoggerFactory.getLogger(MaskUtils.class);

    /**
     * List<object> 打码
     *
     * @param obj
     * @param <E>
     * @return
     * @throws InstantiationException
     * @throws IllegalAccessException
     */
    public static <E> List<E> maskListObject(List<E> obj) throws InstantiationException, IllegalAccessException
    {
        if(null == obj || obj.isEmpty())
        {
            return new ArrayList<>();
        }
        List<E> maskList = new ArrayList<>();
        for(E e : obj)
        {
            maskObject(e);
            maskList.add(e);
        }

        return maskList;

    }

    /**
     * 打码
     *
     * @param obj
     * @param <E>
     * @return
     * @throws IllegalAccessException
     * @throws InstantiationException
     */
    public static <E> E maskObject(E obj)
    {
        logger.info("打码前:{}", JsonUtil.toJSONString(obj));
        if(null == obj)
        {
            return null;
        }
        Class aClass = obj.getClass();
        Field[] declaredFields = aClass.getDeclaredFields();

        for(Field field : declaredFields)
        {
            field.setAccessible(true);
            Sensitive annotation = field.getAnnotation(Sensitive.class);
            if(null == annotation || !field.getType().equals(String.class))
            {
                continue;
            }
            if(annotation.mask())
            {

                String value = null;
                try
                {
                    value = (String)field.get(obj);
                }
                catch (IllegalAccessException e)
                {
                    e.printStackTrace();
                }
                if(StringUtils.isEmpty(value))
                {
                    continue;
                }
                int suffix = annotation.suffixLength();
                int prefix = annotation.prefixLength();
                String character = annotation.sensitiveChar();
                try
                {
                    field.set(obj,
                              maskString(value, produceCharacter(value.length() - prefix - suffix, character), prefix,
                                         suffix));
                }
                catch (IllegalAccessException e)
                {
                    e.printStackTrace();
                }
            }

        }

        logger.info("打码后:{}", JsonUtil.toJSONString(obj));
        return obj;
    }

    public static String maskString(String id, String replacement, int prefix, int suffix)
    {
        if(null == id || "".equals(id))
        {
            return "";
        }
        String value = id.replace(id.substring(prefix, id.length() - suffix), replacement);

        return value;
    }

    public static String maskPhone(String mobilePhone)
    {

        return maskString(mobilePhone, "*", 3, 4);
    }

    public static String produceCharacter(int len, String character)
    {
        StringBuilder sb = new StringBuilder();
        for(int i = 0; i < len; i++)
        {
            sb = sb.append(character);
        }
        return sb.toString();

    }

    private MaskUtils()
    {
    }
}


```

工具类里面抛出两个异常,其实可以查看到是不会出现异常的情况的.


测试类:

```

@Data
public class User{
    
    @Sensitive(prefixLength = 3, suffixLength = 4)
    private String userPhone;

}

User user = new User;
user.setUserPhone("13823456789");
MaskUtils.maskObject(user);
//userPhone:138****6789
```

其实Java反射这样用着还是不错的,第一次尝试自己造轮子. 于是当我第一次把轮子给我的小伙伴的时候,他直接怼了我一句:花了一下午搞定这个,你直接重写toString不久可以了?.......


WTF, 还有这种操作?????????

但是后来仔细想想,他的这种可能侵入就很大了,比如我的类toString是输出类的原始信息,按照他这样,那我怎么获取类原始信息.另外每一个类里面都要去这样写,我这种懒人不太适合.
所以~~~~,我就喜欢造轮子.
