---
title: klook-508-problem
date: 2017-05-08 23:27:55
tags:
	- Question
categories: Question

---

## 5月8日问题

### MySQL二进制运算:
mysql可以直接使用位运算。我们现在是否支持多平台`platform`使用的就是位运算的方式。

|web|mobile web|ios|Android|
|:---:|:------:|:---:|:----:|
|1|0|1|1

这种设置的好处是以后扩展起来可以加位就可以了。其实我觉得很蛋疼，可读性真的差，但是想想linux的权限控制系统rwx其实感觉也还可以接受。不过要是位数异常多的话~GG吧。这种其实也有好处，平常设计表中可能需要多条记录，或者用另一个表关联，这种位运算就可以直接在一个表里面做选择了。那么如何操作位运算了？ 只要记着，你想要那个平台，比如`web`，它就是`8`，因为它指代的是二进制的`1000`,如果你想找出支持`web`平台的记录，那么只要用`platform&8`那么就可以找出所有`web`位是`1`的记录。`select * from tbl_aa where platform&8` 千万别写成`select * from tbl_aa where platform=platform&8` 这样的结果，是将platform进行了赋值，将位运算的结果赋值给了platform再去做查找筛选。<!--more-->
补充一些，位运算还有两个运算符号: 或:`|` ;异或；`^`，下面的代码在数据库中会显示成:
```
select 1^4,4^4,4^12,1|4,1&4
```

|1^4|4^4|4^12|0^0|1l4|1&4|
|:---:|:---:|:----:|:----:|:----:|:----:|
|5|0|8|0|5|0|

所有的数字对于计算机来说都是0101，那么`&`它认为的就是相同的位子都是1才是1（可以看成是一条河流有两个闸门，只有两个闸门都打开==1，水才能一直流如江河，不会断流），`|`它只要当有一个1的时候就认为是对的（相当于一个分流闸门，左边可以走，右边可以走，都可以走，走的通就行），`^`它是只有当两个不相同的时候才为1，相同的时候为0(这就是叫人道理了，凡事不要想着一刀切走极端，凡事有好有坏才是1。哈哈)


### MySQL插入的时候max(num)+1
遇到了一种情况，就是排序的时候默认是max+1，之前想过一种就是先将max取出来，然后再插入，这种情况就是要先查一次数据库，最近用golang语言做后台开发，没少因为调用多个接口多次访问数据库被前辈们教育。我也知道他们是为我好，所以尝试改动。所以想着能少执行一次SQL就开始写了。

```
insert into tbl_myname (id,name,priority)
			values (null,?,(
			select case  when max(temp.priority) IS NULL then '1' else max(priority)+1 end from (
			SELECT priority,id FROM tbl_myname
			) temp
			))
```
```
select case  when max(temp.priority) IS NULL then '1' else max(priority)+1 end from (
			SELECT priority,id FROM tbl_myname
			) temp
```
可以看到这里做了两次查询，难道直接`select max(priority)+1`不行么？还真的不行。 如果不做临时表直接选择`max(priority)+1`数据库执行的时候将会报错`You can't specify target table 'tblmyname' for update in FROM clause`。当然这种插入数据库的方式并不推荐，为啥？因为如果高并发的情况下，我相信，肯定会出现priority相同的情况，如果对priority控制的很严的情况那么是不能出现这种情况的。我能想到的是在进入这个方法的时候加锁，尽量避免并发的情况。

### golang枚举操作
golang是没有枚举这种说法的。但是它有一个关键字`iota`

```
func Test_enum(t *testing.T) {
	fmt.Println(ALL)
	fmt.Println(th_TH)
	fmt.Println(ko_KR)
	fmt.Println(zh_TW)
	fmt.Println(zh_CN)
	fmt.Println(en_US)

	//fmt.Println(ALL|b|c|d)
}

const (
	//英语，中文简体，繁体，韩文，泰文,ALL
	ALL   = iota //0
	th_TH 			//1
	ko_KR 			//2
	zh_TW="test" //3
	zh_CN			// 值为test， iota为 4
	en_US=iota   //5
)

```
用了之后发现，跟我想的不一样。之前在Java里面我可以定义一个value，然后用enum.name()获取名字，就相当于一个key-value键值对，而golang里面貌似这样的做法没有。不知道是是什么原因，不过此路不同总有一条其它的路能通的。

### golang 判断是否为数字
golang里面一开始以为可以直接找一个包或者类来判断传入的字符是否是数字，一开始以为可以用`unicode.IsDigit(r rune)`,后来发现rune其实是一个unit32类型，byte就是一个unit8类型，我的想法仅仅是一个接口然后直接传入string，看来是不行了。所以就想了个注意，自己写。怎么写？第一个想的就是正则：

```
func isDigit(str string) bool {
	if len(str) > 0 {
		var regexp = regexp.MustCompile("^\\d+$")

		matchResult := regexp.MatchString(str)

		if matchResult {
			return true
		} else {
			return false
		}
	}

	return false
}

```

简单方便，直接传入string，就能得到想要的值了？以前Java写的顺手，因为很完善，做业务很方便，如今用go，很多东西也不是很熟悉，所以自己动手丰衣足食。


-------
ps:今天还看到了Java Integer源码里面一个sizeTable
```
final static int [] sizeTable = { 9, 99, 999, 9999, 99999, 999999, 9999999,
                                      99999999, 999999999, Integer.MAX_VALUE };
```
觉得好可爱的写法。哈哈判断位数倒是可以了嘿嘿。
