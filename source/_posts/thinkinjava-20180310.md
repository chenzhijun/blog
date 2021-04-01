---
title: 重温Java编程思想的一些感悟-20180310
date: 2018-03-10 21:54:14
tags: Java
categories: Java
---

# 重温Java编程思想的一些感悟-20180310

> 人的懒惰，是思想上的懒惰

## 前言

今天说个有趣的事情，起床的时间是9点。周末9点起床其实也不算过分，但是吧。拖着拖着在床上又赖到了10点。然后找不到理由再拖下去了。321起床，洗脸刷牙。然后脑袋在思考了：现在要是去大学城吧，10点了。这过去就得半小时吧，然后吃午饭吧，花半小时吧，然后去图书馆路上走个20分钟吧，嗯，好像差不多就12点了。这样下去吧，晚上6点又要吃饭。等会中午说不定还要睡觉。那还去图书馆干嘛。在家里也一样的啊，反正一张大桌子，一个人，很安静。那我先去吃个肠粉当早午饭，吃完回来看书。嗯，出门记得把垃圾扔出去。嗯是的。故事就是这样开始的。当我拿着垃圾扔到楼下垃圾桶。看着太阳这么明媚，天气这么暖和，不知咋地，我竟然生出一种感觉，回家拿书包，去图书馆，而且一分钟也没有思考其它的。直接就又返回来。拿着书包，就出去了。吃完肠粉就去图书馆了。
最后得出一个结论：人啊，要想战胜懒惰，先的开始行动。

今日看了《Java编程思想》的多态，接口，内部类。总共花费蕃茄钟5.5个，当然这只是有效的蕃茄中，天知道我分心走神了几次。。。

## 多态

多态，继承，抽象统称为《面向对象三大核心系统》。多态传统上的用法有点像：基类的方法调用，子类的方法实现。它的作用就是消除类型之间的耦合关系。
了解多态就得先了解下**绑定**，绑定分为前期绑定和后期绑定。

前期绑定：默认的绑定方式，方法直接调用，在编译期就能知道。

后期绑定：根据运行时的对象的类型进行绑定。又称为动态绑定或者运行时绑定。

Java中除了static方法和final方法之外其它都是后期绑定。（private方法是final方法）。只有普通方法的调用是多态的，静态方法不能使用多态的。构造器实际上是static方法，不过是隐式的。酒席那个private的方法是final一样，也是隐式的。

动态绑定只有在运行时才知道，因为无法知道它是属于方法所在的那个类，还是属于那个类的子类。

编写构造器的一条准则：尽可能简单的方法是对象进入正常状态，如果可以的话，避免去调用其它方法。

使用继承还是使用组合？推荐使用组合。

```java
class Actor{
    public void act(){}
}

class HappyActor extends Actor{
    public void act(){
        System.out.print("Happy Actor");
    }
}

class SadActor extends Actor{
    public void act(){
        System.out.print("Sad Actor");
    }
}

class Stage{
    private Actor actor = new HappyActor();

    public void change(){
        actor = new SadActor();
    }

    public void performPlay(){
        actor.act();
    }
}

public class Demo{
    public static void main(String[] args){
        Stage stage = new Stage();
        stage.performPlay();
        stage.change();
        stage.performPlay();
    }
}
```

代码用到了**状态模式**，使用组合还是继承有一条通用准则：用继承表达行为间的差异，并用字段表达状态上的变化。代码中，通过继承获得了两个不同的类，用于表达act()方法的差异。stage通过使用组合的方式通过change()方法试自己的状态发生改变。状态的改变也产生了行为的改变。

一个子类向上转型成父类，总是安全的。但是父类向子类转型就不一定了，毕竟子类的实现可能有很多，直接强转可能会出错（ClassCastException）。

<!--more-->

## 接口

接口的好处有很多，我最大的体会就是，接口中定义方法，子类中各自实现。在项目中我也看过部分代码常量是写在接口中的，工具类是用abstract标明。这种写法我在《effective java》里面看到好像是不推荐这么干的。算了扯回来接口。

谈接口就有点类似抽象类。接口可以看作抽象类的抽象。接口中只有方法定义，新版jdk8支持带默认的方法体了好像要用default。但是实际中，我没用过。。接口中的方法都是public的。接口中声明的域都是static和final的。

策略模式：

```java
class Processor{
    public String name(){
        return getClass().getSimpleName();
    }
    Object process(Object input){
        return input;
    }
}

class Upcase extends Processor{
    String process(Object input){
        return ((String)input).toUpperCase();
    }
}

class Downcase extends Processor{
    String process(Object input){
        return ((String)input).toLowerCase();
    }
}

class Splitter extends Processor{
    String process(Object input){
        return Arrays.toString(((String)input).split(" "));
    }
}

public class Apply{
    public static void process(Processor p,Object s){
        System.out.print("Using processor"+p.name());
        System.out.print(p.process(s));
    }
    private static String s = "Disagreement with beliefs is by definition incorrect";

    public static void main(String[] args){
        process(new Upcase(),s);
        process(new Downcase(),s);
        process(new Splitter(),s);
    }
}

```

**根据所传递的参数对象的不同而具有不同行为的方法**。Processor对象是一个策略。上面的例子是类版本，改成接口版本：

```java
public interface Processor{
    String name();
    Object process(Object input);
}

public abstract class StringProcessor implements Processor{
    public String name(){
        return getClass().getSimpleName();
    }
    public  abstract String process(Object input);
    public static String s = "Disagreement with beliefs is by definition incorrect";

    public static void main(String[] args){
        process(new Upcase(),s);
        process(new Downcase(),s);
        process(new Splitter(),s);
    }
}

class Upcase implements StringProcessor{
    String process(Object input){
        return ((String)input).toUpperCase();
    }
}

class Downcase implements StringProcessor{
    String process(Object input){
        return ((String)input).toLowerCase();
    }
}

class Splitter implements StringProcessor{
    String process(Object input){
        return Arrays.toString(((String)input).split(" "));
    }
}

```

适配器模式：

```java
    class FilterAdapter implements Processor{
        Filter filter;
        public FilterAdapter(Filter filter){
            this.filter = filter;
        }
        public String name(){return filter.name():}
        public Waveform process(Object input){
            return filter.process((Waveform)input);
        }
    }

    public class Filter Processor{
        public static void main(String[] args){
            Waveform w = new Waveform();
            Apply.process(new FilterAdapter(new LowPass(1.0)),w);
            Apply.process(new FilterAdapter(new HighPass(2.0)),w);
            Apply.process(new FilterAdapter(new BandPass(2.0)),w);
        }
    }
```

接口是可以多继承的，使用关键字`extends`。接口也可以用来实现工厂方法设计模式：

```java
interface Service{
    void method1();
    void method2();
}

interface ServiceFactory{
    Service getService();
}

class Implementation1 implements Service{
    Implementation1(){}
    public void method1(){
        System.out.print("Implementation1 method1");
    }
    public void method2(){
        System.out.print("Implementation1 method2");
    }
}

class Implemetation1Factory implements ServiceFactory{
    public Service getService(){
        return new Implementation1();
    }
}

class Implementation2 implements Service{
    Implementation2(){}
    public void method1(){
        System.out.print("Implementation2 method1");
    }
    public void method2(){
        System.out.print("Implementation2 method2");
    }
}

class Implemetation2Factory implements ServiceFactory{
    public Service getService(){
        return new Implementation2();
    }
}

public class Factories{
    public static void serviceConsumer(ServiceFactory fact){
        Service s = fact.getService();
        s.method1();
        s.method2();
    }
    public static void main(String[] args){
        serviceConsumer(new Implemention1Factory());
        serviceConsumer(new Implemention2Factory());
    }
}
```

## 内部类

将一个类的定义放在另一个类的定义内部，这就是内部类。普通内部类，只能放在类的外部层次，不能有static字段。匿名类必须传进必须是final的。

内部类实现工厂方法：

```java
interface Service{
    void method1();
    void method2();
}

interface ServiceFactory{
    Service getService();
}

class Implementation1 implements Service{
    Implementation1(){}
    public void method1(){
        System.out.print("Implementation1 method1");
    }
    public void method2(){
        System.out.print("Implementation1 method2");
    }
    public static ServiceFactory(){
        new ServiceFactory(){
            public Service getService(){
                return new Implementation1();
            }
        }
    }
}

class Implementation2 implements Service{
    Implementation1(){}
    public void method1(){
        System.out.print("Implementation2 method1");
    }
    public void method2(){
        System.out.print("Implementation2 method2");
    }
    public static ServiceFactory(){
        new ServiceFactory(){
            public Service getService(){
                return new Implementation2();
            }
        }
    }
}

public class Factories{
    public static void serviceConsumer(ServiceFactory fact){
        Service s = fact.getService();
        s.method1();
        s.method2();
    }
    public static void main(String[] args){
        serviceConsumer(Implemention1.factory);
        serviceConsumer(Implemention2.facotry);
    }
}
```