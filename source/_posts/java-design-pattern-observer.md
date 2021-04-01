---
title: Java设计模式-观察者模式
date: 2017-09-14 15:22:00
tags:
   - Java
   - 设计模式
categories: Java
---

## 设计模式-观察者模式

### 简述

观察者模式的有点类似现在的手机消息推送，比如你注册了一个APP，然后app有了更新一定会通知你，不然你看看手机里面的推送栏是不是每天淘宝都会推送给你一条消息。哈哈。当然为什么它知道要给你推送，不给哪些非智能机推送了？简单的更类似与现在报纸-订阅。只要你订了报纸，那么每天报纸更新都会把报纸给你送过来。所以我认为观察者模式应该也叫"订阅模式"，因为必须客户端先去注册（订阅）然后才能收到注册中心的更新或者通知。


### 实例

我们先来想想观察者模式中有哪些主体： 

1. 注册中心（主题（或者叫被观察对象））
2. 客户端 (观察者)

现在我们分析下里面的行为：

报社（注册中心-主题）怎么知道有哪些人订了报纸（客户注册）了？那么就是报社必须要有一个记录有哪些人（客户端-观察者）在报社注册的'花名册'；那么现在是法治社会，当然不能强迫去订报纸吧，所以人们订报纸都是自愿的，那么自愿就必须自己去报社订阅(主动注册)。订了报纸之后，不能收不到报纸吧？那报社现在都是比较强势的，一般把报纸送到门口，所以你必须有一个接受的地址，这种算是约定；既然可以订阅，那么当然也有取消订阅（取消注册）；那么报社也就在'花名册'把你的名字去掉就可以了。

整个实现过程就是这样，观察者模式并不复杂，主要是报社(注册中心-主题)，花名册(订阅者名单)，订阅者（接受通知者），订阅(行为)，门(注册中心通知接受者的地址)。
<!--more-->
开始写我们的代码：

```java
    public class NewsPaperSubject{
        private List observers;//花名册

        private String newsPaper; //新闻内容

        public void register(Observer observer){
                observers.add(observer);//注册一个观察者、增加一个订阅客户
        }

        public void unRegister(Observer observer){
            observers.remove(observer);//取消订阅
        }

        public void sendNewsPage(){
            for(Observer obj:observers){
                obj.updateNewsPageOnDoor(newsPaper); // 告知订阅者，新闻有更新
            }
        }

        public void setNews(String news){
            this.newsPaper = news;  // 设置新闻
            sendNewsPage();
        }

   }

   public class Observer{
       private NewsPaperSubject newsPaperSubject;

       public void updateNewsPageOnDoor(String news){
           System.out.println("更新news...."+news);
       }

       public Observer(NewsPaperSubject newsPaperSubject){
            this.newsPaperSubject = newsPaperSubject;
            this.newsPaperSubject.register(this);   
       }
   }
```

根据上面两个类我想你能基本了解订阅者模式是怎样一回事了。不过我们写的代码还是有点问题，耦合太严重，我们不应该针对具体实现编程，所以我们进行一次改版：


```java

    //主题超类
    public interface Subject{
        void registerObserver(Observer observer);
        void removeObserver(Observer observer);
        void notifyObservers();
    }

    public class NewsPaperSubject extends Subject{
        private List observers;//花名册

        private String newsPaper; //新闻内容

        @Override
        public void registerObserver(Observer observer){
                observers.add(observer);//注册一个观察者、增加一个订阅客户
        }

         @Override
        public void removeObserver(Observer observer){
            observers.remove(observer);//取消订阅
        }

        @Override
        public void notifyObservers(){
            for(Observer obj:observers){
                obj.update(newsPaper); // 告知订阅者，新闻有更新
            }
        }

        public void setNews(String news){
            this.newsPaper = news;  // 设置新闻
            notifyObservers();
        }

   }

   // 观察者接口
   public interface Observer{
        void update(String newsPaper);
    }

    // 具体观察者
    public interface StudentObserver{
        private Subject newsPaperSubject;

        public StudentObserver(Subject newsPaperSubject){
            this.newsPaperSubject = newsPaperSubject;
            newsPaperSubject.registerObserver(this);
        }

        @Override
        public void update(String news){
            System.out.println("updateNews.."+news);
        }
    }

```

新版本的情况如果我们有了个政府观察者，我们只需要实现一个Observer新的实现对象就可以了。这种就是针对接口编程，具体的实现细节都是隐藏的。这样其实我们就做到了松耦合。对于注册中心来说只要订阅者实现了Observer的update接口，并且注册到花名单就行了。这样当注册中心发现改变需要通知的时候就可以通知到所有花名单中的订阅者。


