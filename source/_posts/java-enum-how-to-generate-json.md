---
title: Java Enum 与 Json 的互相转换
copyright: true
date: 2019-07-06 00:24:55
tags: 
 - Java
 - Enum
 - Json
categories: Java
---

# Java Enum 与 Json 的互相转换

在Java中使用Enum的频率很高，我们也经常使用 Enum 作为类的一个属性定义。那么如果需要将Enum转换成Json或者将Json传转换成Enum该怎么操作了？接下来我们看一下。Enum序列化成Json的几种方式。

> 本实例使用的是`jackson`的包，用的是`ObjectMapper`.

首先我们定义一个类`Instance`，里面有一个`Enum`的参数: **state** ; 在这里我们为了方便观察几种不同 Enum 的json序列化方式，state定义为`Object`,`Instance`类定义如下;

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
        this.bookName="《Java枚举类-json转换》";
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

<!--more-->

## 原生Enum

这种我们通常是不做任何改动，就一个普通的定义Enum:

```java
package me.chenzhijun.enumjson;

public enum State {
    SUCCESS("success", 1), FAILED("failed", 0);

    private String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public int getNum() {
        return num;
    }

    public void setNum(int num) {
        this.num = num;
    }

    private int num;

    State(String value, int num) {
        this.value = value;
        this.num = num;
    }


}
```

在这种方式下，我们可以看一下转换成json后的输出：

```java
    @Test
    public void enumJsonTest() throws JsonProcessingException {
        //枚举类不做任何改动json转化结果
        System.out.println("枚举不做任何改动json转化结果:");
        Instance instance = new Instance(State.SUCCESS);
        System.out.println(objectMapper.writeValueAsString(instance));
    }
```

运行之后可以看到结果为：

```json
{"state":"SUCCESS","name":"《Java枚举类-json》"}
```

在这种情况下，Enum默认使用的是`enum.name()`来作为json的值。

## 使用 Enum 的所有属性值作为json值

有时候我们可以让Enum像一个普通类一样，json序列化的时候将其中的所有属性都能输出，可以使用的方式是在Enum的定义上加上`@JsonFormat(shape = JsonFormat.Shape.OBJECT)`，代码如下：

```java
package me.chenzhijun.enumjson;

import com.fasterxml.jackson.annotation.JsonFormat;

@JsonFormat(shape = JsonFormat.Shape.OBJECT)
public enum State2 {
    SUCCESS("success", 1), FAILED("failed", 0);

    private String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public int getNum() {
        return num;
    }

    public void setNum(int num) {
        this.num = num;
    }

    private int num;

    State2(String value, int num) {
        this.value = value;
        this.num = num;
    }


}
```

测试方法如上,输出的结果为：

```json
{
    "state": {
        "value": "success",
        "num": 1
    },
    "book_name": "《Java枚举类-json转换》"
}
```

## 使用Enum的某一个属性作为json值

有时候我们可能想要使用Enum中定义的某一个自定义属性的值，只需要在属性的`get`方法上使用`@JsonValue`即可满足需求。

```java
package me.chenzhijun.enumjson;

import com.fasterxml.jackson.annotation.JsonValue;

public enum State1 {
    SUCCESS("success", 1), FAILED("failed", 0);

    private String value;

    @JsonValue
    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public int getNum() {
        return num;
    }

    public void setNum(int num) {
        this.num = num;
    }

    private int num;

    State1(String value, int num) {
        this.value = value;
        this.num = num;
    }


}

```

测试方法如上,输出的结果为：

```json
{
    "state": "success",
    "book_name": "《Java枚举类-json转换》"
}
```

可以看到与原生的Enum相比，state的值变成了我们在Enum中定义的`value`。**注意值的大小写**。

## 自定义序列化结果

有时候我们还可能想更高级一点，那么就可以自定义序列化结果。比如Enum的属性`num`，原本是`int`类型，但是我们就是想让它变成一个`String`类型，或者我不想要Enum其中的某一个变量。那么可以自己实现序列化接口：

```java

package me.chenzhijun.enumjson;

import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.ser.std.StdSerializer;

import java.io.IOException;

public class StateSerializer extends StdSerializer {

    protected StateSerializer() {
        super(State3.class);
    }

    @Override
    public void serialize(Object o, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {

        if (!(o instanceof State3)) {
            return;
        }
        State3 state = (State3) o;

        jsonGenerator.writeStartObject();
        jsonGenerator.writeFieldName("name");
        jsonGenerator.writeString(state.name());
        jsonGenerator.writeFieldName("value");
        jsonGenerator.writeString(state.getValue());
        jsonGenerator.writeFieldName("num");
//        jsonGenerator.writeNumber(state.getNum());
        jsonGenerator.writeString(String.valueOf(state.getNum()));
        jsonGenerator.writeEndObject();
    }
}
```

这样我们定义了我们自己的特殊要求，之后再Enum的定义中进行指定,使用`@JsonSerialize(using = StateSerializer.class)`：

```java
package me.chenzhijun.enumjson;


import com.fasterxml.jackson.databind.annotation.JsonSerialize;

@JsonSerialize(using = StateSerializer.class)
public enum State3 {
    SUCCESS("success", 1), FAILED("failed", 0);

    private String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public int getNum() {
        return num;
    }

    public void setNum(int num) {
        this.num = num;
    }

    private int num;

    State3(String value, int num) {
        this.value = value;
        this.num = num;
    }


}
```

测试方法如上，输出的结果为：

```json
{
    "state": {
        "name": "SUCCESS",
        "value": "success",
        "num": "1"
    },
    "book_name": "《Java枚举类-json转换》"
}
```

附上所有的测试方法：

```java
package me.chenzhijun.enumjson;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.Test;

import java.io.IOException;

public class EnumJsonTest {
    ObjectMapper objectMapper = new ObjectMapper();

    @Test
    public void enumJsonTest() throws IOException {
        //枚举类不做任何改动json转化结果
        System.out.println("枚举不做任何改动json转化结果:");
        Instance instance = new Instance(State.SUCCESS);
        System.out.println(objectMapper.writeValueAsString(instance));//{"state":"SUCCESS","name":"《Java枚举类-json》"}

        System.out.println("\n使用枚举的某一个参数作为json的转化结果:");
        instance = new Instance(State1.SUCCESS);
        System.out.println(objectMapper.writeValueAsString(instance));//{"state":"success","book_name":"《Java枚举类-json转换》"}

        System.out.println("\n将Enum所有的参数一起作为json的转化结果:");
        instance = new Instance(State2.SUCCESS);
        System.out.println(objectMapper.writeValueAsString(instance));//{"state":{"value":"success","num":1},"book_name":"《Java枚举类-json转换》"}

        System.out.println("\n使用serializer自定义enum的json转化结果:");
        instance = new Instance(State3.SUCCESS);
        System.out.println(objectMapper.writeValueAsString(instance));//{"state":{"name":"SUCCESS","value":"success","num":"1"},"book_name":"《Java枚举类-json转换》"}


        System.out.println("json->class");
        String json = "{\"state\":{\"value\":\"success\",\"num\":1},\"book_name\":\"《Java枚举类-json转换》\"}\n";
        instance = objectMapper.readValue(json, Instance.class);
        System.out.println(instance.getState());

    }


}

```

这样，Enum的Json转化就可以任君"宰割"了~~

如果是Json串转Java类型，就把上面的方式换过来即可。比如上一个测试类中的：

```json
        String json = "{\"state\":{\"value\":\"success\",\"num\":1},\"book_name\":\"《Java枚举类-json转换》\"}\n";
        instance = objectMapper.readValue(json, Instance.class);
        System.out.println(instance.getState());
```

嗯嗯~全文完~