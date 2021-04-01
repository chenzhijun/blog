---
title: 使用Apache POI操作生成Execl
date: 2017-12-11 15:17:02
tags:
	- Java
categories: Java
---

## 使用Apache POI操作生成Execl

使用Java操作数据生成Excel表格，在网上能搜到的方式有很多，比如jxl，还有今天用到POI。POI是Apache开源的一个项目，该工具使用简单，方便。

### 准备与一些概念约定

需要在[poi官网](http://poi.apache.org/)下载相应的jar包，或者使用maven来引入包。在开始之前我们需要知道一些关于excel文档的概念。一个excel文件通常称为[workbook]，excel里面的一个工作页面通常叫做[sheet]，sheet里面的格子称为[单元格cell]，单元格由坐标确定唯一位置，相应为[行row]，[列col]。

![2017-12-11-15-41-40](/images/qiniu/2017-12-11-15-41-40.png)

了解这些我们可以开始看看poi的一些类和接口。
<!--more-->
### POI

POI 中主要的几个类为：`HSSFWorkbook`，`HSSFSheet`，`Row`，`Cell`。就像我们创建一个最简单的报表文件一样，先创建excel文件(HSSFWorkbook)，然后创建一个工作页(HSSFSheet)，然后找到哪一行(Row)，在哪一列上创建一个单元格(Cell)。你在excel中操作的最小单位都是Cell，所以我们进行读取和操作的最小单位也是Cell。

嗯，如果你还喜欢猜测，为什么猜测了？因为这几个类都是HSSF开头，那么是不是后面的就是他们的父类，而这些HSSF开头的都是他们的实现了。答案是的。<!--你没猜错，就是具体的实现类，我们为啥用实现来操作，而不是面向接口编程，建议在正式实现的时候用面向接口编程。而我这里就不那么严格了。-->

开始编程前，请记住，你再操作excel的一个步骤，写代码的时候流程也是这样的。

```java
        //创建 excel workbook
        HSSFWorkbook hssfWorkbook = new HSSFWorkbook();
        //创建工作簿 sheet
        HSSFSheet hssfSheet = hssfWorkbook.createSheet();
        //找到需要操作的行row,从第0行开始。我们在excel中看到的行是从1开始数，但是在poi中是从0开始。
        int rowNum = 0;
        Row row1 = hssfSheet.createRow(rowNum);
        //创建单元格cell
        int colNum = 0;
        Cell row1Col1 = row1.createCell(colNum);
        //操作单元格内容
        row1Col1.setCellValue("test");
        // 导出excel文件
        FileOutputStream fout = null;
        try
        {
            fout = new FileOutputStream("test.xsl");
            hssfWorkbook.write(fout);
            fout.close();
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
```

这样一个简单的单元格内容操作就完成了。

### 多变的需求

很多时候我们会有一些特殊的要求，比如文字居中，多行合并，多列合并，加底色，文字颜色等等。。

用的最多的就是文字居中了，这类需求往往都是对于一个单元格有相应要求，POI针对这种需求有一个类：`Style`。用style来设置单元格的相关样式，下面来看一个文本居中的样式代码：

```java
    HSSFCellStyle style = excel.createCellStyle();
    style.setAlignment(CellStyle.ALIGN_CENTER);
    row1Col1.setCellStyle(style);//内容居中显示
```

我们在进行合并的时候，其实实际上合并的还是单元格。所以我们在创建单元格的时候进行一些单元格范围参数设置：

```java
    //单元格范围 参数（int firstRow, int lastRow, int firstCol, int lastCol)
    CellRangeAddress cellRangeAddress = new CellRangeAddress(0, 1, 0, 1);
    //在sheet里增加合并单元格
    hssfSheet.addMergedRegion(cellRangeAddress);
    //生成第一行
    Row row = hssfSheet.createRow(0);
    Cell first = row.createCell(0);
    first.setCellValue("表头");
```

此时被合并的行的其中的单元格都将无效，也就是说再操作这些被合并的单元格都已经不再称为单元格。

```java
    //单元格范围 参数（int firstRow, int lastRow, int firstCol, int lastCol)
    CellRangeAddress cellRangeAddress = new CellRangeAddress(0, 1, 0, 1);
    //在sheet里增加合并单元格
    hssfSheet.addMergedRegion(cellRangeAddress);
    //生成第一行
    Row row = hssfSheet.createRow(0);
    Cell first = row.createCell(0);
    first.setCellValue("first");
    //操作被合并的单元格
    Row row2 = hssfSheet.createRow(1);
    Cell second = row2.createCell(0);
    second.setCellValue("second");
```

像上述代码，second不会输出到excel，因为它的所在单元格(1,0)已经被合并了。

通常我们都会自己写一些工具类，然后进行操作excel操作，你也可以试着自己写一个。我这里就不贴我的代码了。

纸上得来终觉浅，绝知此事要躬行


