---
title: idea-linux-vim等常用快捷键
copyright: true
date: 2018-03-26 20:24:56
tags: All
categories: All
---

最近经常看到一些有意思的快捷键，但是又不是经常用到，平常经常用的肯定都能很熟悉了。

## idea 常用快捷键

最常用的肯定是 find action :`ctrl shift A`，这简直就是神器，如果忘记快捷键，尝试使用这个，然后输入快捷键的功能名称就有可能找到相应 功能了。

1. idea 弹出当前类里面的方法框：`ctrl o`, 类结构图，`file structure`：`ctrl F12`

2. 跳转： navigate -> back/forward; navigate -> last edit location/next edit location

3. 最近的文件：recent file/recent changed file    `ctrl + E`

    <!--more-->

4. 书签跳转，bookmarks : 
    `F11`快速打一个书签；
    `ctrl F11`定一个数字，之后使用ctrl+数字快速定位；
    `shift F11` 弹出书签预览。

5. 收藏： add to favorites, 快捷键: `alt shift F`，你可以先建立一个favorites列表(add new favorites lists)，然后在代码行处使用add to favorites。之后可以使用`alt 2`调用面板查看

6. 跳到某个具体类 `Ctrl N`

7. 找文件：`Ctrl shift N`

8. 找单个字符：`Ctrl shift alt n`,可以在文件中寻找一个单词，字符等。

9. 字符串搜索：Edit->Find->Find in Path: `ctrl shift F`,可以改建，我的改成了`ctrl shift Y`

10. 移动操作：`move caret XXXXX`，然后选择相应的选项

11. 大小写：edit-> toggle case ，`ctrl shift u`

12. 相同字符串多列操作：`select all occurences` --> `ctrl shift alt j`，比如下图中将`=`号右边部分全部加上双引号`""`，这种情况就可以尝试多列操作。

![2018-04-04-09-56-31](/images/qiniu/2018-04-04-09-56-31.png)

在上图中如果只是多列操作，那么后面中文的字符长度不一样，那么不能直接移动，所以可以使用第10条移动操作结合起来一起操作。简直神器。

13. 不知道怎么操作时神奇键：show intention action，`alt enter`

14. 重构：`shift F6`,`ctrl shift alt t`

15. 重构方法，签名等：`ctrl F6`

16. 抽取变量，函数等，`refactor->Extract->xxx`；可以选局部变量，全局变量等。下面是选择的variable

![2018-04-04-11-11-06](/images/qiniu/2018-04-04-11-11-06.png)

![2018-04-04-11-11-41](/images/qiniu/2018-04-04-11-11-41.png)

![2018-04-04-11-11-56](/images/qiniu/2018-04-04-11-11-56.png)


17. 代码最后一次提交人：annotate。
18. 文件修改位置：previous change
19. 版本撤销：revert
20. 本地修改记录：show history
21. 打本地标签：put label，类似 commit
22. 在当前目录新建文件：`ctrl alt insert`
23. 文件复制：`F5`
24. 文件移动：`F6`
25. 复制文件全路径：`ctrl shift C`
26. 复制文件名：`ctrl shift alt C`
27. 选择复制的历史数据：`ctrl shift V`
28. maven 依赖图，右键maven->show dependencies：`ctrl shift alt u`
29. 查看当前类的继承关系，hierarchy class ：`ctrl H`
30. 方法调用，call hierarchy，`ctrl alt H`
## idea 神奇操作：

### live templates

注意live templates 可以使用`$END$`作为最后结束时，光标的位置：

![live tempaltes 实例操作](/images/livetemplate.gif)

### postfix

postfix 是idea预置的，无法增加，使用`ctrl shift a`输入“postfix”就能看到相应的预置postfix。

比如我们想生成下面的代码：

```java
    if(args != null){
        //xxxx
    }
```

使用postfix可能只需要输入:

```java
    args.nn
```

就能出现提示，生成上面的代码

![postfix 实例操作](/images/postfix.gif)

### idea debug 操作

1. 断点： `toggle line breakpoint`：`ctrl F8`。

2. debug模式启动：`shift F9`

3. 单步运行：F8

4. 结束当前断点：resume，F9

5. 关停所有断点：`mute breakpoints`

6. 条件断点：`ctrl shift f8`

7. 表达式求值：`evaluate expression`：`alt F8`

8. 从断点跳到光标行：`run to cursor`：`alt f9`

9. 运行过程中改变值：`set value`，在Debug视图中选中需要改变的变量按`F2`

10. 在当前位置运行，适用与单元测试：`ctrl shift f9`

11. 从历史运行中寻找一个运行：`alt shift f9`-->`Run-->Debug...`

## idea 常用插件

`key promoter`，
`idea vim`：`:sp`
`lombok plugin`，
`maven helper`，
`sonar lint`，
`alibaba java code guide`
`emacsidea`: 使用`ctrl + J` 然后再输入想查找的字符，就可以快速定位了，在keymap中修改acejumpworld。

## VScode 快捷键

`ctrl k` 之后按下 `v`，打开实时预览。