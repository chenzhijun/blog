---
title: Java设计模式-装饰者模式
date: 2017-09-19 10:00:53
tags:
   - Java
   - 设计模式
categories: Java
---

## 设计模式-装饰者模式

> 组合比继承更具有灵活性

### 简述

装饰者模式其实一开始我没想明白，怎么给装饰法？就像当初看Java IO包，始终也没明白怎么这么多类啊。很多人都说它是什么什么装饰者设计模式，那个时候也不怎么懂，就走马观花般过了就过了。
最近重新回到基础，又买了本设计模式的书，其实真的不想看，但是看了一两章发现，我靠，这不就是我之前的搞法么？猛回首，发现自己踩过的坑人家也踩过，而我还在踩。当然能力也有限，记下此博客，一，如果有人能看到这里有点了解，那是甚好；二，如果以后自己忘记了，也有个标记，让我返回至此，然后来个回忆。

回到正途，装饰者，啥事装饰者？看看平常我们说装饰房屋，那么是不是先有个房子，然后再给房子搞下装饰，刷墙啊，放书架啊，这就是一个装饰。那么装饰者模式怎么回事了？*动态的将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的扩展方案。* 装饰者模式的最大特点是在装饰类里面进行了组合，组合的威力发挥到最大。当然这种组合也必须要有一个约束，那么就是必须在同一类中，总不能指驴为马吧。所以装饰的是一类或者一组类似的对象。

### 自我的理解

有时候光想真的会发现自己很多地方好像已经明白了，但是实际上，当我们写代码的时候会发现，要注意点细节真的很多。
<!--more-->
装饰者模式中，其实讲究的是子类对父类的继承扩展，以及子类对于父类子集同类的组合。理解起来其实就是，我可以用我爸爸的人脉帮我办事，也可以用我哥哥的人脉办事，最终我做事可能就不需要太多的人脉了。这种方式很好的解耦了，而且对于扩展的话，我可以随便改，而对于我老爸，老哥，我却没有改的可能。这就是设计模式的一个设计原则：开放封闭原则，类对扩展开放，对修改关闭。

![装饰者模式类图](/images/qiniu/2017-09-19-10-01-02.png)

我们来实际敲敲代码，假设我们要开咖啡店，咖啡店有多种原料，根据不同的原料组成不同的咖啡，拥有不同的售价。

![装饰者实际类图](/images/qiniu/2017-09-19-10-02-16.png)

首先我们建立基础类：


```java

package me.chenzhijun.coffee.type;

/**
 * 咖啡的基类
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public abstract class Beverage {
    String description = "unknown beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();

}

```

之后在咖啡基础类上建立具体的咖啡：

```java

package me.chenzhijun.coffee.type;

/**
 * 焦炒咖啡
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public class DarkRoast extends Beverage{
    public DarkRoast(){
        description = "DarkRoast...";
    }
    @Override
    public double cost() {
        return 1.1;
    }
}


package me.chenzhijun.coffee.type;

/**
 * 浓缩咖啡
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public class Espresso extends Beverage {
    public Espresso(){
        description = "espresso...";
    }

    @Override
    public double cost() {
        return 1.2;
    }
}

package me.chenzhijun.coffee.type;

/**
 * 优选咖啡
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public class HouseBlend extends Beverage {
    public HouseBlend() {
        description = "houseblend...";
    }

    @Override
    public double cost() {
        return 0.1;
    }
}

```

可以看到上面的三个咖啡都是继承自抽象类Beverage，形成`同类`；

接下来我们建立装饰者的基类：

```java

package me.chenzhijun.coffee;

import me.chenzhijun.coffee.type.Beverage;

/**
 * 装饰者
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public abstract class CondimentDecorator extends Beverage {
    public abstract String getDescription();
}

```

看到装饰者类也是继承Beverage，并且也同样是抽象类，唯一的抽象方法就是要实现getDescription();

之后我们在装饰者类的基础上增加几个具体的装饰者：

```java

package me.chenzhijun.coffee;
import me.chenzhijun.coffee.type.Beverage;

/**
 * 摩卡咖啡
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public class Mocha extends CondimentDecorator {
    Beverage beverage;

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription()+" mocha...";
    }

    @Override
    public double cost() {
        return beverage.cost()+3.1;
    }
}

package me.chenzhijun.coffee;
import me.chenzhijun.coffee.type.Beverage;

/**
 * 大豆奶咖啡
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public class Soy extends CondimentDecorator {
    Beverage beverage;

    public Soy(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + " soy...";
    }

    @Override
    public double cost() {
        return beverage.cost() + 1.1;
    }
}

package me.chenzhijun.coffee;
import me.chenzhijun.coffee.type.Beverage;

/**
 * 加奶泡的咖啡
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public class Whip extends CondimentDecorator {

    Beverage beverage;

    public Whip(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + " whip...";
    }

    @Override
    public double cost() {
        return beverage.cost() + 2.1;
    }
}


```

现在我们队咖啡加了一些装饰：摩卡，大豆，奶泡，这些都是装饰者类，而且装饰者类里面都有一个`Beverage`,这就方便我们组合其他的同族的类的功能。

最后来看先卖咖啡：

```java
package me.chenzhijun.coffee;

import me.chenzhijun.coffee.type.Beverage;
import me.chenzhijun.coffee.type.DarkRoast;
import me.chenzhijun.coffee.type.Espresso;
import me.chenzhijun.coffee.type.HouseBlend;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/19
 */
public class Starbucks {
    public static void main(String[] args) {
        Beverage beverage = new Espresso();//浓缩咖啡已经是成品了

        System.out.println(beverage.getDescription() + ", " + beverage.cost());

        //双份摩卡加奶泡
        Beverage beverage2 = new DarkRoast();
        beverage2 = new Mocha(beverage2);
        beverage2 = new Mocha(beverage2);//多一份摩卡
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription() + ", " + beverage2.cost());

        // 大豆+摩卡+奶泡
        Beverage beverage3 = new HouseBlend();
        beverage3 = new Soy(beverage3);//
        beverage3 = new Mocha(beverage3);
        beverage3 = new Whip(beverage3);

        System.out.println(beverage3.getDescription() + ", " + beverage3.cost());

    }
}
```

这样就可以看到如果我们单纯的采用继承，那么将会继承自多个类，而且很笨重，而组合的方式很灵活；

![功能主要调用图](/images/qiniu/2017-09-19-14-03-12.png)

装饰者模式中，jdk中用的最多的可能就是IO包了。基本上所有的IO类都继承至四大主类:`Reader/Writer;InputStream/OutputStream`。

其实我们只要理解了一个其它的就是可以类比了。 

> Reader/Writer 是在Stream之后引进的，因为对字符的操作比较多，所以引进了新的字符输入输出。jdk1.7之后都实现了AutoCloseable接口,try-with-resources语法来实现自动关闭流。

我们仅仅来看下InputStream类，所有的输入流都可以算作是InputStream的子类，在InputStream中定义了几个方法，查看子类中有个FilterInputStream类，而且大多数的类都继承自FilterInputStream而不是InputStream，仔细的看下FilterInputStream类中的方法，可以发现FilterInputStream中有一个InputStream的属性，而所有的方法都是用的是这个属性的方法，FilterInputStream就是一个装饰类，查看它的一个子类DataInputStream，可以发现DataInputStream中只有一个构造方法为DataInputStream(InputStream);FilterInputStream类的说明中，也指到它的作用就是包含一个其它的InputStream族类，然后使用它们的增加的方法来为己用。

类比下装饰者模式，FilterInputStream可以说就是一个装饰者，我们要用哪个InputStream的时候也是将InputStream层层包括装饰进去。如果这样来看JavaIO流其实也就并没有那么复杂了。当然因为装饰者模式的特点，我们可以看到生产了很多的小类，是的IO包的类异常的多。


参考文档：
Head First 设计模式 第三章装饰者模式