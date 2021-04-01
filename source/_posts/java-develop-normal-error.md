---
title: Java 开发遇到的两个问题
copyright: true
date: 2019-03-31 21:56:43
tags: Java
categories: Java
---

# Java开发遇到的两个问题

## Object 反序列化失败

使用`@ReponseBody`返回一个json串，返回的类型是`Object`，我们知道如果是`@RestController`，都会已restful返回，也就是返回json格式的数据，但是如果你是使用Object返回值，然后Object只是一个null或者仅仅只是`new Object()`，那么就会返回下面的异常：

```log

com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class java.lang.Object and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: java.util.ArrayList[1])

```

## RestTemplate使用中path有{}问题

RestTemplate 默认就是将path中的`{}`作为一个赋值表达式的，它会认为你需要替换`{}`中的内容。所以在path中最好特别注意下。