# 枚举的前身？
    在枚举之前我们通常都是使用final int常量，这样的方式有什么缺点和好处？
    
```java
        public interface Fruit{
            public static final int APPLE = 0; 
            public static final int ORANGE = 1;
            public static final int ORANGE = 1;
            public static final int ORANGE = 1;
            public static final int ORANGE = 1;
            

        }
```

不够直观；打印了0，1，谁知道是什么鬼？

Enum出现了。(**适用与固定值的常量**)

# 如何创建枚举？

枚举继承至Enum，使用关键字**enum**就可以声明枚举了。

# 枚举的使用场景-单列模式

```java
public enum Singleton {
    INSTANCE;

    private Object object;
    private Singleton(){
        // init something..
        object = new Object();
    }

    public void doSomething(){
        //....
    }

    public Object getObject() {
        return object;
    }
}
```

# 枚举的父类Enum？

从Enum中继承;了什么？enum中的clone方法能克隆么？

# 编译器帮我们做了什么？

编译器会自动帮忙做一些事：创建toString()方法，equals(), hashCode(), ordinal()方法，static values(),valueOf(String) 返回的值顺序是定义的时候的顺序

# 枚举在使用中的注意点？

1. 避免使用ordinary方法，强依赖于定义顺序，添加了之后会有变化，可以加域
2. 枚举适用于有限集合，值是确认的。有限的集合适用于switch。
3. 枚举构造方法是private
4. 所有枚举值都是public , static , final的
5. 除了不能继承之外，可以将enum看成一个常规类，为什么不能继承？Javap..  不能继承，但是可以实现接口
6. 定义枚举不能声明为final，static
7. 不能在定义完之后作为类来定义一个新实例。Clone是final 保证了不能重写，反射是被禁止的，枚举特殊化的序列化机制确保了反序列化一样

# 参考资料：

[枚举Enum 的常用方法](https://www.cnblogs.com/yxh1008/p/6538756.html)
[责任链模式](http://blog.csdn.net/fangfengzhen115/article/details/50272113)
[枚举序列化](http://www.baeldung.com/jackson-serialize-enums)
[jls8 8.9 Enum Types]