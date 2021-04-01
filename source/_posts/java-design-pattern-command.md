---
title: Java设计模式-命令模式
date: 2017-10-04 16:18:05
tags:
   - Java
   - 设计模式
categories: Java
---

## 设计模式-命令模式

命令模式是啥啊?感觉从字面上来理解,就是命令-执行.那么谁下命令?谁执行?执行的人从哪里知道命令?又怎么知道要执行哪些命令? 
__命令模式的定义: 将'请求'封装成对象,以便使用不同的请求,队列或者日志来参数化其它对象.命令模式也支持可撤销的操作__

### 命令模式

让我们假设一件事情,如果有个遥控器,上面有个按钮,按钮可以控制家里的一件东西,比如说开关门,开关灯,开关电视.那么我们该怎么做了?
其实这个就相当于一个命令模式了.人是下命令的.也就是按钮执行就是一个命令,按钮对应了什么操作,按下的操作,这个操作怎么执行了?是门执行,还是灯去执行,还是电视了? 
如果可以,我们可以想想另外一个例子:下馆子.通常我们去到餐馆,服务员给个菜单,然后我们写上我们要吃的菜;完事之后服务员拿走菜单,然后告诉厨师,厨师拿到菜单后开始炒菜.
这个过程中,"我们"就相当于一个客户端,我们将菜品(请求)封装成了一个订单对象.服务员拿到请求,告诉厨师请求来了,厨师拿到订单看到里面的菜品(命令)就开始执行.
<!--more-->
命令模式过程图:
![2017-10-04-16-21-22](/images/qiniu/2017-10-04-16-21-22.png)

类关系图
![2017-10-04-20-47-21](/images/qiniu/2017-10-04-20-47-21.png)


这个过程如果想清楚了,回到我们为遥控器编程,现在我们拿到的就是一个空遥控器,要控制那个东西也不明白,所以让我们尝试编程看.

### 命令编程

我们从简单的开始,必不可少的是命令的执行者,我们假设这次我们控制的是灯泡,`Light.java`:
```java
//命令执行者,命令执行对象
public class Light {
    public void on() {
        System.out.println("light is on...");
    }
}
```

然后我们想要让它执行命令,想想看在订餐的时候,菜单是不是都是一样的? 所以我们先定义一个命令标记接口`Command.java`:
```java
/**
 * 命令接口
 */
public interface Command {
    /**
     * 命令接口具体执行
     */
    public void execute();
}
```

之后灯的开关,是不是执行一个命令,我们先只管"灯开"这一个命令(暂时不管灯关),这个命令肯定是继承命令标记接口的,因为我们肯定是要先统一了菜谱,然后再让客户去点菜的.所以我们的灯开命令`LightOnCommand.java`:
```java
/**
 * 灯开的命令
 */
public class LightOnCommand implements Command {
    // 命令执行者
    Light light;

    //命令执行方法
    @Override
    public void execute() {
        light.on();
    }

    /**
     * 设置命令执行者
     * @param light
     */
    public LightOnCommand(Light light) {
        this.light = light;
    }
}
```

命令对象也有了,我们现在来让遥控器适配上,我们建立一个简单遥控器类`SimpleRemoteControl.java`:
```java
/**
 * 遥控器
 */
public class SimpleRemoteControl {
    // 命令执行
    Command slot;

    public SimpleRemoteControl() {
    }

    // 设置按钮命令,以后也可以绑定灯的开关,或者门的开关.
    public void setSlot(Command slot) {
        this.slot = slot;
    }
    //遥控器按钮按下去
    public void buttonWasPressed(){
        slot.execute();
    }
}
```

这些做完之后,我们看看,遥控器里面我们组合的是一个命令,而命令又统一继承自一个接口,然后各个命令的实现里面又各个命令执行者的组合实例.这样来看,我们按下遥控器,遥控器就会执行命令,命令会知道是那个执行者去执行.现在编写测试类`RemoteControlTest.java`:
```java
//测试类
public class RemoteControlTest {
    public static void main(String[] args) {
        //遥控器
        SimpleRemoteControl remote = new SimpleRemoteControl();

        //电灯
        Light light = new Light();
        LightOnCommand lightOnCommand = new LightOnCommand(light);

        //命令设置
        remote.setSlot(lightOnCommand);
        //命令执行
        remote.buttonWasPressed();

    }
}
```

这样,我们对遥控器编程算是完成了.如果我们想控制门,或者控制电视,那我们该怎么弄?其实就模仿灯的方式就行了.在这个实例里面,我们可以看到,遥控器其实是不知道具体是让灯开,还是让门开的,就是遥控器的按钮命令是可以任意的,这个应该算是解耦了把.遥控器只知道命令只要实现了`Command`的接口就可以了.

命令模式感觉这样确实非常清晰的,代码的demo版本理解和实现也不困难,但是不知道在实际开发中,怎么去用它,将它用在些设呢么地方.我在电商系统中的购物车下单是否可以这样使用了? 但是貌似也就一个地方用到了下单,那么我直接实现是否会比使用设计模式会更好了? 现在我感觉我的问题是,不知道怎么去扩展,没地方扩展,自然就会觉得简单直接的方式最好,也就无需去考虑适设计模式.但是设计模式应该在什么地方,什么时候去使用了?真是路,从来就没有一条好走的路.