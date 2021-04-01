---
title: Java IO流
date: 2017-09-20 16:35:43
tags:
	- Java
categories: Java
---

## Java IO流

### Java IO流简述

JavaIO 中流都分别在java.io包中，主要分为几类：

* 字节流：Byte Stream->InputStream/OutputStream
* 字符流: Character Stream->Reader/Writer
* 缓冲流：Buffered Stream
* 数据流：Data Stream 只针对基本数据类型：(boolean, char, byte, short, int, long, float, double) 还有String
* 对象流: Object Stream,继承自Serializable,必须序列化
* 控制台的流..

### InputStream

字节流其实又叫做二进制流，因为它是最基本的二进制的数据的读取，也就是010101的读取与传输。如果读取的数据全是文字类型，那么推荐使用字符流的，如果是图片或者其它的图片可以使用字节流。看下`InputStream` 类里面的相关方法：

![2017-09-20-17-24-13](/images/qiniu/2017-09-20-17-24-13.png)

InputStream实现了Closeble,Closeble继承了AutoCloseable，其实就是相当与一个标记，AutoCloseable是1.7才出来的，主要是为了`t-w-r`语法主动关闭流。

InputStream是`abstract`的，一定要注意它是一个抽象类。我们说的装饰者模式里面，InputStream的实际实现(实现了InputStream的相关方法)基本都是类似于组件，而InputStream相当与一个超类。里面的主要构成是8个方法一个属性。

#### available()

```java
    public int available() throws IOException {
        return 0;
    }
```

先看我们的第一个方法`available()`,它的作用是返回流中估计的长度，而这个长度怎么设置？看源码中的介绍最后归结两句话：

```
1. 永远也不要调用此方法来确认分配多大的缓存空间，因为它永远返回0。
2. 任何继承`InputStream`的子类必须重写该方法。
```

意思就是子类自己去实现这个方法。
<!-- more -->

#### mark()

```
public synchronized void mark(int readlimit) {}
```

`mark()`方法在原文中有个：` Marking a closed stream should not have any effect on the stream.` 这里的方法是个空方法，子类需要自己实现，当然这里没有强制要求。只是mark的作用是在reset的时候开始的。当调用mark的时候会从readlimit的位置开始将之后读到的内容放到缓存块中。这样reset之后再读取就能取到值了。

#### markSupported()

`markSupported()`方法是指是否支持`mark`和`reset`方法，只有两者都支持得时候才能返回`true`,默认为`false`;
可以查看`FileInputStream`和`DataInputStream`,这两个子类有分别不同的方式。

#### read()

```java
   public abstract int read() throws IOException;
```

读取流中的下一个字节数，返回一个0-255的int值。如果读到流的最后一个字节，返回`-1`。该方法子类必须自己重写。

#### read(byte[])

```java
public int read(byte b[]) throws IOException {
        return read(b, 0, b.length);
    }
```

将流的内容读取到`b`字节数组（缓冲数组）中。如果b的长度为0，那么不会有任何的字节会读取到数组中，并且返回0。如果不是的话，它会将缓存小于等于b的数组长度，一次一次来读取内容。读取到最后返回-1。


#### read()

```java

    public int read(byte b[], int off, int len) throws IOException {
        if (b == null) {
            throw new NullPointerException();
        } else if (off < 0 || len < 0 || len > b.length - off) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return 0;
        }

        int c = read();
        if (c == -1) {
            return -1;
        }
        b[off] = (byte)c;

        int i = 1;
        try {
            for (; i < len ; i++) {
                c = read();
                if (c == -1) {
                    break;
                }
                b[off + i] = (byte)c;
            }
        } catch (IOException ee) {
        }
        return i;
    }

```

其实这个方法可以看到是调用的`read()`方法来实现的。目的就是将读到的`byte`字节一个一个缓存到字节数组中。

#### reset()

```java
    public synchronized void reset() throws IOException {
        throw new IOException("mark/reset not supported");
    }
```

通俗的讲如果你在书的某一个位置放了一个书签，那么在你想返回到书签的位置的时候。这个动作就叫做reset().
默认是抛出异常的，除非子类自己实现它。调用reset的时候，markSupport也必须返回true。并且mark的位置不能是无效的。比如你放书签不能超过书的范围吧。

#### skip(long)

```java
    public long skip(long n) throws IOException {

        long remaining = n;
        int nr;

        if (n <= 0) {
            return 0;
        }

        int size = (int)Math.min(MAX_SKIP_BUFFER_SIZE, remaining);
        byte[] skipBuffer = new byte[size];
        while (remaining > 0) {
            nr = read(skipBuffer, 0, (int)Math.min(size, remaining));
            if (nr < 0) {
                break;
            }
            remaining -= nr;
        }

        return n - remaining;
    }
```

直接看代码实现。里面使用的read(byte,0,length);这不就是读取字符么。跳过多少个字符，就是先读多少个字符，读完之后之后读的字符再返回给你，返回实际跳过的字符。

####  MAX_SKIP_BUFFER_SIZE

`private static final int MAX_SKIP_BUFFER_SIZE = 2048;` 这个属性的默认值为2048，好像也就只在skip里面用到了，就是定义一个最大的缓存大小。

### 其它流

所有的字节流都是继承InputStream。

非缓存流获取数据是直接跟OS打交道。缓存流将数据读到内存空间然后内存空间空了才去调用底层OS方法。自动刷缓存可以设置`autoflush`,通过构造方法设置。

建立一个缓存流`new DataOutputStream(new BufferedOutputStream(new FileOutputStream(dataFile)))`

Data stream : DataInputStream/DataOutputStream  

> 不要使用浮点数来表示精确的数字，如果需要精确的数字用BigDecimal

Object Streams : ObjectInputStream/ObjectOutputStream 

### 文件流（NIO）