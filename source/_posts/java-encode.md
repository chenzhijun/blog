---
title: Java-encode 编码注意事项
date: 2017-05-10 23:10:01
tags:
	- Java
	- Encode
categories: Java
---

Tomcat URL 编码配置
http://localhost:8080/examples/servlets/servlet/饺子?author=饺子

其中
http: 对应scheme，协议

localhost: 对应Domain，主域

8080: port，端口，配置在tomcat<Connector port="8080"/>

examples: ContextPath，配置在tomcat<Context path="/examples"/> 

servlet/servlet: ServletPath，在web.xml中<url-pattern>中配置

饺子: pathInfo，指到具体的servlet

author=饺子: QueryString，传递的参数，如果是post就是表单方式提交

浏览器编码将非ASCII字符编码成16进制数然后再之前加上"%"
<!--more-->
对URL的URI不分进行解码的字符实在<Connect URIEncoding="UTF-8">中定义的.默认为ISO-8859-1，QueryString的解码字符要么是Header中的ContentType定义的charset，默认是ISO-8859-1，要使用ContentType中定义的编码，需要将connector的<Connector URIEncoding="UTF-8" useBodyEncodingForURI="true"/>，其中的useBodyEncodingForURI 仅仅是对QueryString进行BodyEncoding解码，不针对整个URI。

