---
title: Java设计模式-工厂模式
date: 2017-10-04 13:06:49
tags:
   - Java
   - 设计模式
categories: Java
---

## 工厂模式

最近看了很久的工产模式，一直没敢下笔。因为我发现我就算弄了很久也就明白了简单工厂和工厂模式，抽象工厂一直没有弄的太明白。今天想着不如试试看，至少先把自己会的记录下来吧，不然越久越容易忘记了。

__工厂模式定义:定义了一个创建对象的接口,但由子类决定实例化的具体对象是哪个.工厂方法让类把实例化推迟到了子类中.__

### 工厂简述

想想一个例子，假如我们想吃批萨，那么如果是简单的话，我们可能就买个面粉自己揉几下然后放到烤箱里面。只要出来的时候是个饼的形状，也就差不多了（对我这种不调食的人来说...）。但是我们可能还有一些有追求的人，他可能想吃海鲜批萨，可能想吃芝士批萨，可能想吃原味批萨。这么多要求，你觉得在家里自己做麻烦么？麻烦怎么办？丢个别人呗。那么我们就建个工厂，专门给我们做批萨，而我想吃什么批萨的时候，打个电话过去，告诉它给我做个什么口味的批萨，然后它把吃的给我送过来就好了。这种方式就基本上就是一个工厂模式了。
很显然，我们将"做批萨解耦出来了。工厂负责做批萨，而我只要批萨就行，至于你批萨是怎么做的，你工厂随便弄，最后给个批萨给我，就行。
<!--more-->

### 简单工厂

其实简单工厂是工厂模式的一种特例，严格来说它就是工厂模式。我们先来看看有哪些东西我们要在实例中用的：批萨，吃批萨的人或者说需要批萨的客户端，生成批萨的工厂。

产品批萨，如果各个地方不同，可以实现该类。`Pizza.java`:

```java
public class Pizza {
    private String pizzaName;
    private String cheese;

    public String getCheese(){
        return cheese;
    }
    public String setCheese(String cheese){
        this.cheese = cheese;
    }

    public String getPizzaName() {
        return pizzaName;
    }

    public void setPizzaName(String pizzaName) {
        this.pizzaName = pizzaName;
    }

    public Pizza() {
    }

    public Pizza(String pizzaName) {
        this.pizzaName = pizzaName;
    }

    public void prepare() {
        System.out.println("prepare...");
    }

    public void bake() {
        System.out.println("bake...");
    }

    public void box() {
        System.out.println("box...");
    }
}
```

简单批萨工厂，`SimplePizzaFactory.java`:

```java
public class SimplePizzaFactory {
    public Pizza createPizza(String pizzaType) {

        Pizza pizza = null;

        if (pizzaType.equals("cheese")) { // 芝士批萨
            pizza = new Pizza("cheese");
        }else if(pizzaType.equals("chicken")) {//鸡肉批萨
            pizza = new Pizza("chicken");
        }
        return pizza;

    }
}
```

需要批萨的人，我们将它看成批萨店好了:`PizzaStore.java`:

```java
public class PizzaStore {

    private SimplePizzaFactory factory;

    public PizzaStore(SimplePizzaFactory factory) {
        this.factory = factory;
    }

    public Pizza orderPizza(String pizzaType) {

        Pizza pizza = factory.createPizza(pizzaType);
        pizza.prepare();
        pizza.bake();
        pizza.box();
        return pizza;

    }
}
```

现在如果有人加盟店，继承自PizzaStore，然后传入他们的工厂就行了。比如说我想在深圳开一个深圳批萨，然后深圳本地有个当地的批萨工厂。

```java
public class ShenZhenPizzaStore extends PizzaStore{
    public static void main(String[] args){
        SimplePizzaFactory simplePizzaFactory = new ShenzhenPizzaFactory();
        PizzaStore pizzaStore = new PizzaStore(simplePizzaFactory);
        pizzaSore.orderPizza("chicken");
    }
}
```

这样任何在深圳地方的人想吃批萨只要在批萨店调用`orderPizza`方法就行了。

不过想想看，如果有一天我们有个长沙批萨店，他们要特殊化，主打高端人士批萨，这种时候他可能就希望是自己来决定是自己来生产批萨还是找长沙的批萨工厂。这种时候怎么办了？我们将代码做些改进，既然不管那个加盟店都必须要继承自`PizzaSore`,那我们将PizzaStore做一些改动：

```java
public abstract class PizzaStore {
    public Pizza orderPizza(String pizzaType){
        Pizza pizza = createPizza(pizzaType);
        pizza.prepare();
        pizza.bake();
        pizza.box();
        return pizza;
    }
    protected abstract Pizza createPizza(String pizzaType);

}

```
Pizza
```java
public abstract class Pizza {
    private String pizzaName;

    public String getPizzaName() {
        return pizzaName;
    }

    public void setPizzaName(String pizzaName) {
        this.pizzaName = pizzaName;
    }

    public void prepare() {
        System.out.println("prepare...");
    }

    public void bake() {
        System.out.println("bake...");
    }

    public void box() {
        System.out.println("box...");
    }
}

public class ChangShaPizza extends Pizza{
    public ChangShaPizza(){
        name = "ChangSha";
    }
}
```

然后长沙的pizza店：
```java
public class ChangShaPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String pizzaType) {
        if (pizzaType.equals("ChangSha")) {
            return new ChangShaPizza();
        }
        return null;
    }
}
```

我们将它定义为虚拟抽象类，并且增加抽象方法。这样每一个继承它的子类都必须自己实现一次`createPizza`方法，这样子类的实现，父类就完全不管了，只要定个统一接口就好了。

该总结一下啦，修改后我们的类主要是：PizzaStore,Pizza。我们将他们分下类：创建者类和产品类。

`PizzaStore` ：可以相当于创建者(Creator)类，它是一个抽象方法，定义了一个抽象工厂方法，让子类实现该方法来制造产品。创建者里面也会有抽象的产品(这里相当于pizza，因为你也可以继承pizza然后实现自己的独特pizza)的依赖，不过创建具体哪种产品交由子类实现。我们在`ChangShaPizzaStore`里面就是实现了具体的产品。

`Pizza` ： 相当于抽象产品类，其实我们也可以实现我们自己的Pizza，然后给它加点形状，特色啥的。这是个相当于工厂生产的产品抽象.如下图所示:
![2017-10-04-14-51-34](2017-10-04-14-51-34.png)

其中Creator就相当于我们的PizzaStore所有的加盟店必须继承它,并且实现创建pizza的方法,而ConcreteCreator是唯一知道应该创建那个一个具体产品的(ChangShaPizza),所有的产品都实现一个产品接口Product.

可以看到如果加盟店越多,可能pizza的具体种类也就越多,而我们的PizzaStore只依赖于Pizza这个顶级产品类,而具体是ChangShaPizza还是BeijingPizza它就不需要管那么多了.这里有个原则可以引出来:__依赖倒置原则:要依赖抽象,而不依赖具体实现类__.

### 抽象工产模式

__抽象工厂模式:提供一个接口,用于创建相关或依赖对象的家族,而不需要明确指定具体类.__

![2017-10-04-15-40-38](/images/qiniu/2017-10-04-15-40-38.png)

可以看到,我们将工厂创建批萨的工厂也用一个工产接口来实现,而我们的新的store只依赖于工厂接口而不再是任何一个具体工厂接口.

如何理解了? 比如工厂模式中,我们生产pizza是基于子类的具体实现的,子类里面去进行具体的pizza封装,然后进行实例化,而抽象工产,它是不会去创建具体实例对象的,它的职责是创建相关或依赖对象的家族,它的每一个接口都是创建的一个单独的对象.比如Pizza,它会去创建做一个pizza需要的各种原料,各个原料其实都是一个单独的对象个体,它称为一个原料工厂.原料工厂是一个接口,然后各个实现了该接口的具体类去具体实现每种原来如何创建.

```java
public class BeijingPizza extends Pizza{
    PizzaIngredientFactory pizzaIngredientFactory;

    public BeijingPizza(PizzaIngredientFactory pizzaIngredientFactory){
        this.pizzaIngredientFactory = pizzaIngredientFactory;
    }

    void prepare(){
        cheese =  pizzaIngredientFactory.createCheese();
    }
} 
```

工厂模式和抽象工厂模式,他们之前一个是用的继承方式(工厂模式),一个是用的组合(抽象工厂).

抽象工厂: 如果需要创建产品家族和想让制造的想关产品集合起来的时候可以用抽象工厂.

工厂模式: 把客户端代码从需要实例化的具体类中解耦.

总之工厂就是将对象的创建封装了起来,更加的松耦合,做到弹性的设计.