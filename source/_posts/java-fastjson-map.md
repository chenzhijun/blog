---
title: FastJson 转换Map问题
date: 2017-08-19 20:38:07
tags:
	- Java
categories: Java
---

## 使用FastJson 转换成HashMap<String,List<Object>>问题

### 背景需求

有个json串需要转换成Map<String,List<Object>>这种格式,在使用fastjson的时候发现出现异常,调试后发现是类型转化出错,它将List<Object>转成了JSONArray<JSONObject>,这个就很尴尬了.这个问题也找了很久,一直以为它可以直接转换成我想要的List<Object>,没想到最后来了个JSONArray.


### 解决方案
<!--more-->
因为实际中我们需要的是List而不是JSONArray,如果每个都是自己手动去改变,不太符合程序员的特质.所以想到一个通用办法解决:将JSONArray的值取出来,转换成List再重新塞进Map对象里面,具体实现代码:

```
    private static <T> HashMap<String, List<T>> fromJson2Map(String jsonString)
    {
        HashMap<String, Object> jsonMap = JSON.parseObject(jsonString, HashMap.class);

        HashMap<String, List<T>> resultMap = new HashMap<>();
        for(String key : jsonMap.keySet())
        {
            JSONArray jsonArray = (JSONArray)jsonMap.get(key);
            List list = handleJSONArray(jsonArray);
            resultMap.put(key, list);
        }
        return resultMap;
    }

    private static <T> List<T> handleJSONArray(JSONArray jsonArray)
    {
        List<T> list = new ArrayList();
        for(Object object : jsonArray)
        {
            JSONObject jsonObject = (JSONObject)object;
            T obj = (T)JSONObject.parseObject(jsonObject.toJSONString(), Object.class);
            list.add(obj);
        }
        return list;
    }
```

在最后`T obj = (T)JSONObject.parseObject(jsonObject.toJSONString(), Object.class);` 其实我一开始是犹豫的,主要是Object.class,后来想想Java的Object是一切对象的父类,自然可以多态转成子类啊.所以这里就完美的解决了问题. 当然代码应该会有性能问题,应该是new ArrayList() 吧, 每次都会new一个新对象,用完了之后也没有其它的用了,一直等着垃圾回收?.突然觉得自己路还有很长的要走.