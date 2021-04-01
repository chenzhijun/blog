---
title: Java设计模式-策略模式
date: 2017-09-12 15:22:23
tags:
   - Java
   - 设计模式
categories: Java
---

## Java设计模式-策略模式

### 简述

策略，什么是策略？就在不同情况下用不同的方式去处理。那么策略模式是什么？ 模式，模式简而言之就是套路，就是前人总结下来得到验证的套路。所以策略模式其实就是用策略去解决一类事情的套路。


### 策略模式的一个小思考

说到套路，那么一个套路就要有组成部分，不然怎么套路？而策略模式中，它的主要点就是变化的情况，以及应对变化的处理方法。比如有一家超市打折，可能为了吸引顾客、庆祝节日等，但是可能中途来个产品经理，他说为了利润最大化，要根据付款总数来对顾客实行不同打折额度。这种时候程序员心中都是千万匹马儿在奔腾，就不能为为别人做点好事么？（其实心里就是想着，tmd，又改需求，我还想回家陪老婆孩子的）~~~咳咳，走远了。 就这个超市打折来说，打折这种事情是一定存在的，而对打折的方式可能还会有各种变化。那么怎么去应对这种变化？再说一个例子，如果有个制造鸭子的工厂，我们都知道鸭子有很多类（比如死的，活的），死的就像玩具类~，活的就像烤鸭类(貌似烤之前才是活的.),不管怎样，我想说大多数鸭子都有一些基本行为，比如：飞？叫？游泳？诸如此类..但是我们会发现，鸭厂产出的鸭子最终还是要卖出去的，那么这个鸭子能不能飞？能不能叫？能不能游泳？那都是买家说了算，谁给钱听谁的。那么好了A可能要一个能飞，但不能叫，也不能游泳；B呢想要个能飞，能叫，能游泳；这下如果鸭厂只能生产一种只会飞，或者只会叫的鸭子，那肯定是不行的，但是也不能为了A专门搞个生产线，B专门搞个生产线，这样搞，要是N多客户，N多需求，那可怎么办？说了这么多，基本是废话，那么还是来代码看看。

<!--more-->

### 鸭厂造鸭

鸭厂生产鸭,先确定一个`Duck`类，鸭子基本上有飞，叫，游泳的功能属性;那么鸭子就有了三个方法：

```

public abstract class Duck{

    public void fly(){};

    public void swim(){};

    public void gua(){};//呱呱叫
}

```


先看下如果要满足A，不用策略模式的话，我们是这样玩：

```
public class ADuck extends Duck{

        @Override
        public void fly(){
            System.out.println("A的鸭子会飞");
        }

        @Override
        public void swim(){
            System.out.println("A鸭子不能游泳");
        }

        @Override
        public void gua(){
            System.out.println("A鸭子不能叫，太吵");
        }
}
```

对于每一个需求方都需要重新实现一遍，这是不现实的，要是上千个可怎么办？而且用继承的方式，我们就必须的检查每一个继承的方法，那些方法要重写， 那些不需要.

### 鸭厂改造

想想看鸭子，鸭子还是鸭子，我们换个角度想，鸭子了，是不会变的，那它还是主体；客户变的是啥？是鸭子的功能，而这种功能我们可以集中起来，对于鸭子我们将它的功能抽出来。当他需要什么功能再给它什么功能。只有它需要我再给。这也是一个设计原则：**多用组合，少用继承**

```
public interface FlyBehavior{
    void fly();
}

public interface SwimBehavior{
    void swim();
}

public abstract class Duck{
    FlyBehavior flyBehavior;
    SwimBehavior swimBehavior;

    public void fly(){
        flyBehavior.fly();
    };

    public void swim(){
        swimBehavior.swim();
    };

    //public void gua(){};//呱呱叫
}

```

我们用接口将飞和游泳的行为组合进去Duck，你会问，为啥不直接让Duck实现flyBehavior和swimBehavior接口了？其实如果使用实现的话，那么就会发现一个鸭子可能实现了N个行为，而且对于N个行为你都的重写他的实现方法，不能好的复用。

然后我们只需要实现FlyBehavior和SwimBehaior接口就可以了：
```
陆地鸭：

public class CanNotSwim implements swimBehavior
{
    @Override
    public void swim()
    {
        System.out.println("不会游泳的鸭子....");
    }
}

用翅膀飞的鸭子：

public class FlyWithWings implements FlyBehavior
{
    @Override
    public void fly()
    {
        System.out.println("用翅膀飞......");
    }
}
```

然后我们将鸭子的行为给我们要造的鸭子：

```
会飞不回游泳的鸭子：

public class CanFlyButCanNotSwimDuck extends Duck{
    
    public CanFlyButCanNotSwimDuck()
    {
        swimBehavior = new CanNotSwim();
        flyBehavior = new FlyWithWings();
    }
}

```

刚刚是通过构造方法来指定行为，将鸭子造出来的。那么这样其实貌似每一种鸭子我好像还是得new一个子类出来，还是不太优雅，怎么弄？

给鸭子升级：

```
public class Duck{

    public void fly( FlyBehavior flyBehavior){
        flyBehavior.fly();
    };

    public void swim( SwimBehavior swimBehavior){
        swimBehavior.swim();
    };

}


public class Test{
    public static void main([]string args){
        Duck A = new Duck();
        a.fly(new FlyWithWings());
        a.swim(new CanNotSwim());

        Duck B = new Duck();
        B.fly(new FlyWithWings())
    }
}

```

当然也可以通过`set`方法来设置行为。

```
 public class Duck{
    
    FlyBehavior flyBehavior;
    SwimBehavior swimBehavior;

    public void fly(){
        flyBehavior.fly();
    };

    public void swim(){
        swimBehavior.swim();
    };

    public void setFlyBehavior(FlyBehavior flyBehavior){
        this.flyBehavior=flyBehavior;
    }

 }
```

这种时候我们就能看到策略模式的用处了，当来了一个新的功能，比如造一个'带火箭飞的鸭子'，那么只需要实现一个飞的行为就可以了

```

public class FlyWithRocket implements FlyBehavior
{
    @Override
    public void fly()
    {
        System.out.println("火箭带飞....");
    }
}

```

只需要需要的地方再设置成飞的行为为`FlyWithRocket`就可以了。

可以看到我们将会变化的部分抽离出来，然后进行组合；这也说明了软件开发的一个设计原则：**针对接口编程,而不是实现编程**。对于抽出来的一组行为，其实是独立于鸭子的，这也说明我们可以在其他地方使用这些行为。比如造鸭鸣器就可以复用鸭子的叫的行为。

### 策略模式总结

总的来说，策略模式就是定义了一组行为，将各行为分别封装起来，然后可以互相替换。这组行为是独立于真正使用行为的实体的。


>如果在再开发中，思考下哪些是变动的，哪些是可以抽出来的。业务变动是不可预计的，不能为了设计而设计，但是多留心，代码也能写的更加优雅。