---
title: Java 使用 XStream 操作 XML
date: 2017-04-21 22:57:21
tags:
	- Java
	- XStream
categories: Java
---

## Java 使用 XStream 操作 XML

### XStream 简介

XML 是一种严格的文本格式，我想大家都是知道的。XStream 的作用主要是操作 XML，当然JDK也有自己的方法来实现，但是今天我们用XStream来操作XML。
<!--more-->

### XStream 基本使用

XStream 可以将 Object 序列化成 XML。如果有包结构，那么序列化成xml的时候节点会带上包名。

`Student.java`

```java
package com.czj.student;

public class Student {

    private String name;

    private String major;

    private int age;
}
```

我们希望生成的`xml`的文件为:

```xml
<Student>
  <name>小王</name>
  <major>英语专业</major>
  <age>2</age>
</Student>
```

如果直接使用XStream来生成xml如下:

`Main.java`

```java
package com.czj.student;

import com.thoughtworks.xstream.XStream;

/**
 * Created by alvin on 4/21/17.
 */
public class Main {

    public static void main(String[] args) {
        Student student = new Student("小王","英语专业",2);
        Student student2 = new Student("小李","法学专业",3);

        XStream xStream = new XStream();
        String xml = xStream.toXML(student);

        System.out.println(xml);

    }
}

```

运行后生成:`xml`

```xml
<com.czj.student.Student>
  <name>小王</name>
  <major>英语专业</major>
  <age>2</age>
</com.czj.student.Student>
```

可以看到如果是带有包名的Object(Studen.java)，那么生成的xml节点就会带上包名。这明显就不是我们所需要的XML。那么如何改变了？可以用到Annotation注解的形式:

```java
@XStreamAlias("student")
public class Student {
}
```
给所要改名的节点的类上加上`@XStreamAlias("student")`注解，貌似可以了？其实加上了注解之后，那么XStream怎么知道要取解析注解了？所以我们要指明一点：

```java
xStream.processAnnotations(Student.class);
```

所以原来的代码就变成:`Student.java`

```java
package com.czj.student;

import com.thoughtworks.xstream.annotations.XStreamAlias;

/**
 * Created by alvin on 4/21/17.
 */

@XStreamAlias("student")
public class Student {


    private String name;

    private String major;

    private int age;


    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getMajor() {
        return major;
    }

    public void setMajor(String major) {
        this.major = major;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Student() {
    }

    public Student(String name, String major, int age) {
        this.name = name;
        this.major = major;
        this.age = age;
    }
}

```

而`Main.java`:
```java
package com.czj.student;

import com.thoughtworks.xstream.XStream;

/**
 * Created by alvin on 4/21/17.
 */
public class Main {

    public static void main(String[] args) {
        Student student = new Student("小王","英语专业",2);

        XStream xStream = new XStream();
        xStream.processAnnotations(Student.class);//开启注解
        String xml = xStream.toXML(student);

        System.out.println(xml);

    }
}

```

就能生成我们想要的`xml`结果了:

```xml
<student>
  <name>小王</name>
  <major>英语专业</major>
  <age>2</age>
</student>
```

### XStream 操作命名空间

seo中通常要生成站点sitemap.xml.我们尝试去写一个对应的java类:Sitemap.java

```java
package com.czj.sitemap;

import com.thoughtworks.xstream.annotations.XStreamAlias;
import com.thoughtworks.xstream.annotations.XStreamAsAttribute;
import com.thoughtworks.xstream.annotations.XStreamImplicit;

import java.util.List;

/**
 * Created by alvin on 4/21/17.
 */

@XStreamAlias(value="urlset")
public class Sitemap{

    @XStreamImplicit
    private List<Url> url;

    @XStreamImplicit
    private List<ImageUrl> imageUrl;


    @XStreamAsAttribute
    @XStreamAlias("xmlns")
    private String xmlns = "http://www.sitemaps.org/schemas/sitemap/0.9";

    @XStreamAsAttribute
    @XStreamAlias("xmlns:image")
    private String image = "http://www.google.com/schemas/sitemap-image/1.1";


    public List<Url> getUrl() {
        return url;
    }

    public void setUrl(List<Url> url) {
        this.url = url;
    }

    public String getXmlns() {
        return xmlns;
    }

    public void setXmlns(String xmlns) {
        this.xmlns = xmlns;
    }


    public List<ImageUrl> getImageUrl() {
        return imageUrl;
    }

    public void setImageUrl(List<ImageUrl> imageUrl) {
        this.imageUrl = imageUrl;
    }

    public String getImage() {
        return image;
    }

    public void setImage(String image) {
        this.image = image;
    }
}
```

可以看到命名空间

```xml
<?xml version="1.0" encoding="utf-8" ?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
  <url>
    <loc>loc0</loc>
    <changefreq>changefreq0</changefreq>
    <lastmod>lastmod0</lastmod>
    <priority>priority0</priority>
  </url>
</urlset>
```

### XStream 常用注解介绍

`@XStreamAlias(value="urlset")`: 生成的节点别名

`@XStreamImplicit` : 忽略节点生成，常用于List a ，a也是一个节点，而实际上a并不需要。

`@XStreamAsAttribute
    @XStreamAlias("xmlns")` : 属性，属性别名

**千万要记得如果采用注解形式，一定要将`xStream.processAnnotations(Student.class);`打开，否则无法运用注解**









