---
title: java-serializable
date: 2017-12-19 09:26:01
tags:
---

实现序列化的接口必须提供无参构造方法，否则在运行时报错。
反序列化的时候，非序列化类会使用无参构造方法将属性初始化。


the default mechanism for restoring the object's non-static and
 * non-transient fields

  If a serializable class does not explicitly declare a serialVersionUID, then
 * the serialization runtime will calculate a default serialVersionUID value 
 * for that class based on various aspects of the class, as described in the
 * Java(TM) Object Serialization Specification.

 However, it is <em>strongly
 * recommended</em> that all serializable classes explicitly declare
 * serialVersionUID values, since the default serialVersionUID computation is
 * highly sensitive to class details that may vary depending on compiler
 * implementations, and can thus result in unexpected
