---
title: Vim 初步使用
date: 2017-05-09 22:10:01
tags:
	- Vim
categories: Linux
---

# vim使用命令

:help <command>
:set nu
i: 插入
a: 当前光标后面插入
x: 删除当前光标
u: 撤销
control + r  前进
:w  保存
:q  退出
:wq  保存并退出
:q! 强制退出
dd 删除一行并且存入剪切板
2dd 剪切2行,从当前光标行开始计算
p 粘贴
o 之后新增一行
O 之前新增一行
cw 从光标开始到第一个空格全部删除,并在当前位置插入
0   数字0,到行头
$  到行末
<!--more-->

^  到本行第一个不是blank字符的位置(blank指代空格,tab,换行,回车)
g_ 到本行最后一个不是blank字符的位置(blank指代空格,tab,换行,回车)
fa 查找下一个为a的字符处,fs 下一个为s的字符处..
3fa 当前行第3个出现a的位置
ta 查找下一个为a的字符的前一个字符处,ts 下一个为s的字符的前一个字符处..
F T 和 f t 类似,只是方向想反

dt" 删除所有内容,直到遇到"


/patern 文本查找,用n下一个

yy 拷贝
3yy 拷贝三行

:e <path/to/file> 打开一个文件
:saveas <path/to/file> 另存为
:bn :bp 切换打开的文件

.   重复上次的命令
N<command> 重复某个命令多少次
 eg:2dd 删除2行,剪切也是
 	3p  粘贴文本三次
 	10idesu [ESC] 写下10次desu desu desu desu desu desu desu desu desu desu 
 	. 重复上一个命令--10"desu"
 	3. 重复3次"desu".只会输入3个



N G 到几行 比如34G 跳转到34行
gg 到第一行
G 到最后一行
w 到下一个单词的开头 W
e 到下一个单词的结尾 E
* # 当前光标在那个字母上,按*或者#后,再按一次可以让光标移动了,\*是下一个,#是上一个

% 可以匹配(,{,[ 括号的另一半,光标直接移动到另一个位置

`<start position><command><end position>`
命令联动 0y<dollar符号>  复制一行


gU 转大写
gu 转小写
v 视图模式 ,然后按下hjkl 就可以选择了
视图模式下如果有,`(map (+) ("foo"))`.而光标键在第一个 o 的位置
vi" → 会选择 foo.
va" → 会选择 "foo".
vi) → 会选择 "foo".
va) → 会选择("foo").
v2i) → 会选择 map (+) ("foo")
v2a) → 会选择 (map (+) ("foo"))

i和a可以代表in all

块操作
control + v
control + d
Itest [ESC] 
< > 缩进
J 行缩进

:split 分屏
control+w 切换



