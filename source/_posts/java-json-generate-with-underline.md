---
title: Java 类转 Json 时属性名使用下划线
copyright: true
date: 2019-07-06 00:26:28
tags: 
 - Java
 - Json
categories: Java
---

# Java 类转 Json 时属性名使用下划线

很多时候和其它系统对接的时候，不太喜欢用驼峰的方式，毕竟如果是给前端的api，写个驼峰总感觉有点怪异，总喜欢将属性使用下划线。也就是一个`bookName`,给前端的时候是：`book_name`，当然如果是后台系统，还是使用驼峰啊。

## Java 统一 json 为下划线

在 Java 中有两种方式可以实现这种方式，一种是全局的，一种是局部。
<!--more-->
### 全局修改属性的json名为下划线格式

如果需要全局的修改，那么只需要在类上面使用`@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)`即可。

```java
package me.chenzhijun.enumjson;

import com.fasterxml.jackson.databind.PropertyNamingStrategy;
import com.fasterxml.jackson.databind.annotation.JsonNaming;

@JsonNaming(PropertyNamingStrategy.SnakeCaseStrategy.class)
public class Instance {
    Object state;
    private String bookName;

    public Instance(Object state){
        this.state=state;
        this.bookName="《json转换》";
    }

    public Instance() {
    }

    public Object getState() {
        return state;
    }

    public void setState(Object state) {
        this.state = state;
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }
}

```

转换后的json为：

```json
{
    "state": "Success",
    "book_name": "《json转换》"
}
```

### 局部修改属性的json名为下划线格式

局部的方式就是使用`@JsonProperty(value = "book_name")`这个作用在属性上：

```java
package me.chenzhijun.enumjson;

import com.fasterxml.jackson.annotation.JsonProperty;

public class Instance {
    String state;

    @JsonProperty(value = "book_name")
    private String bookName;

    public Instance(String state) {
        this.state = state;
        this.bookName = "《json转换》";
    }

    public Instance() {
    }

    public Object getState() {
        return state;
    }

    public void setState(String state) {
        this.state = state;
    }

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }
}

```

测试类为：
```java
    @Test
    public void jsonTest() throws JsonProcessingException {
        Instance instance = new Instance("success");
        System.out.println(objectMapper.writeValueAsString(instance));
    }
```