---
title: IFE栅格化布局任务相关知识
date: 2016-06-15 19:57:35
tags: CSS
---
#### 实现思路:
1. box-sizing属性
将border和padding计算入width之内,消除两者对于盒子宽度计算的影响.
2. @media媒体查询
根据查询不同浏览器宽度调用不同样式
3. calc方法
动态计算div的width
<!--more-->
-----
#### 1. box-sizing
> box-sizing更符合人们对于"盒子"这个概念的认知,也省去了对于border以及padding的计算,让有边框的盒子可以正常使用百分比宽度

##### box-sizing的值:
1. content-box(_默认_): border和padding不计算入width之内
2. padding-box(_部分浏览器不支持_): padding计算入width内
3. border-box: border和padding计算入width之内

> [Demo地址](http://wkd2015.github.io/Demo/demos/pages/study/boxsizingTest.html)

-----
#### 2. @media媒体查询
> 媒体查询包含了一个媒体类型和至少一个使用如宽度、高度和颜色等媒体属性来限制样式表范围的表达式。CSS3加入的媒体查询使得无需修改内容便可以使样式应用于某些特定的设备范围。


@media规则可以让你在同一张样式表中使用不同的样式来适应不同的媒介

##### 语法
``` bash
<!-- link元素中的CSS媒体查询 --> 
<link rel="stylesheet" media="(max-width: 800px)" href="example.css" /> 
<!-- 样式表中的CSS媒体查询 --> 
<style> 
@media (max-width: 600px) { 
    .facet_sidebar { 
        display: none; 
    } 
} 
</style>
```
当媒体查询为真时,相关的样式就会按照正常的级联规则应用.即使媒体查询结果为假,link查询的样式表仍会被下载.在不使用not和only操作符时,媒体类型是可选的,默认为all

##### 逻辑操作符
1. and操作符: 
把多个媒体属性组合起来,合并到同一条媒体查询中.只有当每个属性都为真时,这条查询的结果才为真
2. not操作符:
用来对一条媒体查询的结果进行取反
3. only操作符
表示仅在媒体查询匹配成功的情况下应用指定样式.可以通过它让选中的样式在老式浏览器中不被应用

> 1. 若使用了not或only操作符,必须明确指定一个媒体类型
> 2. 也可以将多个媒体查询以逗号分隔放在一起.只要其中任何一个为真,整个媒体语句就返回真.相当于or操作符

-----
#### 3. calc()方法
> calc是英文单词calculate(计算)的缩写,是css3的一个新增的功能,用来指定元素的长度.可以使用calc()给元素的border、margin、pading、font-size和width等属性设置动态值


calc()能让你给元素的做计算，你可以给一个div元素，使用百分比、em、px和rem单位值计算出其宽度或者高度

``` bash
.elm {
  width: calc(expression);
}
```

##### calc()的运算规则
1. 使用“+”、“-”、“*” 和 “/”四则运算；
2. 可以使用百分比、px、em、rem等单位；
3. 可以混合使用各种单位进行计算；
4. 表达式中有“+”和“-”时，其前后必须要有空格，如"widht: calc(12%+5em)"这种没有空格的写法是错误的；
5. 表达式中有“*”和“/”时，其前后可以没有空格，但建议留有空格。

> calc()方法转载自w3cplus,更详细内容见[链接](http://www.w3cplus.com/css3/how-to-use-css3-calc-function.html)