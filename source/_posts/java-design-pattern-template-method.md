---
title: Java设计模式-模板方法模式
date: 2017-10-09 20:55:53
tags:
   - Java
   - 设计模式
categories: Java
---

## 模板方法设计模式

### 简述

模板方法模式,它其实肯定是有一个模板的.模板是什么?就是假定的标准.而这个标准由我们定义.模板方法指什么了?它就是封装了一系列操作到一个方法里面.为啥这么说?看生活中的一个实例:

### 生活例子

* 泡咖啡: 如果我们想要喝咖啡,我们会怎么做?
    1. 烧水;
    2. 放咖啡;
    3. 选个马克杯子;
    4. 加点牛奶或糖.

泡一杯咖啡,我们大概就是这样操作.让我们再看看有些人不喜欢咖啡,喜欢喝茶.那么应该是怎样了?

* 泡茶: 如果喜欢喝茶,泡一杯茶怎么做?
    1. 烧水;
    2. 放茶叶;
    3. 选个保温杯?;
    4. 加奶或者加柠檬?.加点奶变奶茶,或者加点柠檬变成柠檬茶?
<!--more-->
泡茶的步骤,我们大概就是这样了.

看看泡茶和泡咖啡的流程,它是不是可以抽象出来?比如说
* 第一步: 烧水,我统一在一个地方烧水不就行了?要烧两次浪费电么(电烧水壶).
* 第二步: 放咖啡/放茶叶,简单来说,这不是就放饮料而已么.
* 第三步: 选杯子,马克杯还是保温杯不就是选一个杯子啊.
* 第四步: 加奶,加糖,加柠檬.总的来说就是加调料.

如果仔细想想看这几个步骤,总体来说他们都是一样的.所以我们抽象出来就是:

1. 烧水
2. 放饮料
3. 选杯子
4. 放调料

如果我们在代码里面实现就是:

饮料模板类:`BeverageTemplate.java`,饮料模板类定义了一系列的模板方法,其中如果是共用的,比如`boilwater()`,那么子类继承之后就无需再单独实现了,另外如果是其它的方法,那么就交由具体的子类单独去实现.

```java 

/**
 * 饮料模板
 * Beverage:饮料
 */
public abstract class BeverageTemplate {

    /**
     *准备饮料
     * 将一系列方法按照一个固定的模板排布
     */
    public void prepareBeverage(){
        boilWater();//烧水
        putBeverage();//放饮料
        packageCup();//选杯子装杯
        addCondiment();//放调料
    }

    private void boilWater() {
        System.out.println("烧开水...");
    }

    protected abstract void addCondiment();

    protected abstract void packageCup();

    protected abstract void putBeverage();
}
```

咖啡类,继承自饮料模板类,并且重写了几个模板方法:

```java
public class CoffeeBeverage extends BeverageTemplate {

    @Override
    protected void addCondiment() {
        System.out.println("加糖,咖啡太苦...");
    }

    @Override
    protected void packageCup() {
        System.out.println("选个星巴克的杯子好拍(zhuang)照(bi)");
    }

    @Override
    protected void putBeverage() {
        System.out.println("放现磨咖啡");
    }
}
```

茶类,继承自饮料模板类,重写父类的方法:

```java

public class TeaBeverage extends BeverageTemplate{
    @Override
    protected void addCondiment() {
        System.out.println("放点奶,变奶茶");
    }

    @Override
    protected void packageCup() {
        System.out.println("选个保温杯,多喝几次");
    }

    @Override
    protected void putBeverage() {
        System.out.println("放茶叶");

    }
}
```

可以看到每一个子类都有自己的具体操作实现,但是他们的操作步骤确已经订好了.这有点类似是,父类只是定义接口,具体实现交个子类.

但是你可能会想,加不加调料,那是喝的人说了算啊.我们现在的定义方式不就是所有的人都要加,那如果我想喝的就是绿茶,不需要奶,怎么办? 这种时候,我们其实可以稍微做些改动,修改一下我们的模板类:

```java
/**
 * 饮料模板
 * Beverage:饮料
 */
public abstract class BeverageTemplate {

    /**
     *准备饮料
     * 将一系列方法按照一个固定的模板排布
     */
    public void prepareBeverage(){
        boilWater();//烧水
        putBeverage();//放饮料
        packageCup();//选杯子装杯

        if(needCondiment()){
            addCondiment();//放调料
        }
    }

    // 是否需要调料
    private boolean needCondiment() {
        return true;
    }

    private void boilWater() {
        System.out.println("烧开水...");
    }

    protected abstract void addCondiment();

    protected abstract void packageCup();

    protected abstract void putBeverage();
}
```
修改后我们可以看到,我们成功的加入了一个钩子,这个钩子有啥用?那就是当我们的子类如果不需要调料的时候,子类重新实现一下即可.我们也可以实现一个空方法如下面:

```java
/**
 * 饮料模板
 * Beverage:饮料
 */
public abstract class BeverageTemplate {

    /**
     *准备饮料
     * 将一系列方法按照一个固定的模板排布
     */
    public void prepareBeverage(){
        boilWater();//烧水
        putBeverage();//放饮料
        packageCup();//选杯子装杯

        if(needCondiment()){
            addCondiment();//放调料
        }
        
        otherMethod();
    }

    protected void otherMethod(){};

    // 是否需要调料
    private boolean needCondiment() {
        return true;
    }

    private void boilWater() {
        System.out.println("烧开水...");
    }

    protected abstract void addCondiment();

    protected abstract void packageCup();

    protected abstract void putBeverage();
}
```
像上面这种,我们加入了`otherMethod()`然后里面实现是一个空方法.这种空方法如果看jdk会发现有很多都是这样弄的.比如io包里面就有.

### 总结
现在是时候来总结一下了.茶和咖啡,我们将通用步骤抽出来,然后放到父类的通用模板里面,而具体的实现操作我们交给子类,子类的各实现中,我们应该不要互相调用.尤其是不应该调用父类的方法.这样的目的是减少依赖.即父类可以调用子类的实现,而子类中的实现应该避免去调用父类的组件.这个有点类似考大学,学子只需要好好读好书,考高分就行了,而学校是否录取你,你填好自愿之后,你也别去问了,它需要的时候自然回来通知你.书中称这种为`好莱坞原则`:别去调用高层次组建,高层次组件回来调用你.

现在该给模板方法模式下定义了:`在一个方法中定义了一个算法的骨架,而将一些步骤延迟到子类中.模板方法使得子类可以在不改变算法结构的情况下,重新定义算放中的某些步骤.`

仔细回想,我们的`prepare()`它可不可以算个算法步骤?第一步干嘛,第二步干嘛..这就是算法骨架么.然后所有的子类都将实现抽象方法.这不就是将实现放到子类中了么.

好了,就到此了.如果你有什么问题,欢迎一起探讨.

参考资料: <<Head First 设计模式>>