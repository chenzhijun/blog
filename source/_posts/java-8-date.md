---
title: Java8中的日期时间
date: 2017-11-29 22:19:56
tags:
	- Java
categories: Java
---

## Java8中的日期时间

最近尝鲜，之前了解过Java8的新日期API，但是一直没有真正的去尝试使用，这次有一个新的项目而且不是特别重要，所以就开始了自己的一次尝试。

### 简介

新的日期API是位于`java.time.*`包下。大致用到的类如下：

![2017-11-29-23-29-34](/images/qiniu/2017-11-29-23-29-34.png)

```
    ZoneId: 时区ID，用来确定Instant和LocalDateTime互相转换的规则
    Instant: 用来表示时间线上的一个点
    LocalDate: 表示没有时区的日期, LocalDate是不可变并且线程安全的
    LocalTime: 表示没有时区的时间, LocalTime是不可变并且线程安全的
    LocalDateTime: 表示没有时区的日期时间, LocalDateTime是不可变并且线程安全的
    Clock: 用于访问当前时刻、日期、时间，用到时区
    Duration: 用秒和纳秒表示时间的数量
```

其中跟日期和时间相关的类为：`LocalDate`,`LocalDateTime`,`LocalTime`;
根据他们的名字也可以看出来`LocalDate`主要针对日期操作，
`LocalDateTime`主要针对日期+时间进行操作，
`LocalTime`主要为针对时间进行操作。
其实对于西方来说，有Date和Time区分的，分为日期类型和时钟类型。

另外跟日期和时间相关的就是时区了：`TimeZone`,`ZoneId`;8中内置了很多时区，可以根据需要选择。

另外一个就是`Instant`，这个指的时间线上的一个点，比如你看成坐标轴的横坐标。

<!--more-->
### LocalDate,LocalDateTime,LocalTime

新的日期API核心三大块。`LocalDate`是指的:`yyyy-MM-dd`，这种类型的日期，比如：2017-11-21。所以你可以猜一下，如果是一个日期工具类，平常我们肯定要用到`LocalDate`，

1. 它作为工具类应该提供静态方法来获取一个对象所以有`LocalDate.now()`来获取一个LocalDate对象;
2. 然后我们有日期了，要获取一个LocalDate对象，那么必须有解析的静态方法：`LocalDate.parse(string)`，
3. 如果想要获取到某一个具体日期的LocalDate对象：`LocalDate.of(int year, Month month, int dayOfMonth)`;
4. 以前的日期api不能很方便的操作日期，如果我们要加一天或者减一天都很麻烦，java8 既然是后来者，肯定要有优化所以有了：`plus()`和`minus()`;
5. 加一年或者一个月呢？`plusYear()`,`plushMonth()`,`plusWeeks()`,`plushDays()`;

有加必有减，所以就可以知道大部分api内容了。然后这三个API的内容基本一样。

**推荐使用LocalDateTime,因为它将日期和时钟都获取了，可以很方便你进行各种日期时间操作**

### 时区

时区这个操作就好说了，直接看代码，因为java提供了一个默认时区，它会根据当前运行时环境自动判断：

```java

    @Test
    public void testDate() {
        System.out.println(ZoneId.systemDefault());// 跟操作系统时区相关

        ZoneId zoneId = ZoneId.of("Asia/Shanghai");//CTT
        System.out.println(zoneId);

        TimeZone timeZone = TimeZone.getTimeZone("CTT");

        System.out.println(timeZone.toZoneId());

        System.out.println("====================");
        for (String id : TimeZone.getAvailableIDs()) {
            System.out.print(id + ",");
        }

        System.out.println("\n====================");
        TimeZone timeZone1 = TimeZone.getTimeZone("Asia/Samarkand");
        System.out.println(timeZone1.toZoneId());
    }
```

![2017-11-29-22-58-54](/images/qiniu/2017-11-29-22-58-54.png)

### Instant

`Instant`，表示的是时间线上的一点，我个人认为主要就是将LocalDateTime和Date新旧API连接起来，他们之间可以如下装换：

LocalDateTime 转成 Date

```java
    Instant instant = LocalDateTime.now().atZone(ZoneId.systemDefault()).toInstant();
    Date date = Date.from(instant);
```

Date 转成LocalDateTime

```java
    LocalDateTime localDateTime = LocalDateTime.from(new Date());
    System.out.println(localDateTime);
```

LocalDate 转成 Date

```java
    Date date =
                Date.from(LocalDate.now().atStartOfDay().atZone(ZoneId.systemDefault()).toInstant
```

### 小技巧

#### 日期格式化

字符串格式：

```java
    LocalDateTime now = LocalDateTime.now();
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    System.out.println("默认格式化: " + now);
    System.out.println("自定义格式化: " + now.format(dateTimeFormatter));
    LocalDateTime localDateTime = LocalDateTime.parse("2017-08-20 15:26:12", dateTimeFormatter);
    System.out.println("字符串转LocalDateTime: " + localDateTime);
```
#### 日期周期

Period类用于修改给定日期或获得的两个日期之间的区别。

给初始化的日期添加5天:

```java
    LocalDate initialDate = LocalDate.parse("2017-07-20");
    LocalDate finalDate = initialDate.plus(Period.ofDays(5));
    System.out.println("初始化日期: " + initialDate);
    System.out.println("加日期之后: " + finalDate);
```

周期API中提供给我们可以比较两个日期的差别，像下面这样获取差距天数:

```java
    long between = ChronoUnit.DAYS.between(initialDate, finalDate);
    System.out.println("天数差: " + between);
```

#### 是否润年

```java
    boolean leapYear = LocalDate.now().isLeapYear();
    System.out.println("是否闰年: " + leapYear);
```

#### 减去一个月

```java
    LocalDate prevMonth = LocalDate.now().minus(1, ChronoUnit.MONTHS);
```

#### 一周的第几天

```java
    DayOfWeek dayOfWeek = LocalDate.now().getDayOfWeek();
    System.out.println("周几: " + dayOfWeek);
    int dayOfMonth = LocalDate.now().getDayOfMonth();
    System.out.println("第几天？: " + dayOfMonth);
```

#### 获取这个月第一天

```java
    LocalDate firstDayOfMonth = LocalDate.now()
                    .with(TemporalAdjusters.firstDayOfMonth());
    System.out.println("这个月的第一天: " + firstDayOfMonth);
    firstDayOfMonth = firstDayOfMonth.withDayOfMonth(1);
    System.out.println("这个月的第一天: " + firstDayOfMonth);
```

#### 判断是否是我的生日

判断今天是否是我的生日，例如我的生日是 1995-03-19

```java
    LocalDate birthday = LocalDate.of(1995, 03, 19);
    MonthDay birthdayMd = MonthDay.of(birthday.getMonth(), birthday.getDayOfMonth());
    MonthDay today = MonthDay.from(LocalDate.now());
    System.out.println("今天是否是我的生日: " + today.equals(birthdayMd));
```

#### 判断是否之前，之后

```java
    boolean notBefore = LocalDate.parse("2017-07-20")
                        .isBefore(LocalDate.parse("2017-07-22"));
    System.out.println("notBefore: " + notBefore);
    boolean isAfter = LocalDate.parse("2017-07-20").isAfter(LocalDate.parse("2017-07-22"));
    System.out.println("isAfter: " + isAfter);
```
