---
title: '关于width:auto和width:100%差异的新理解'
date: 2016-07-19 23:39:45
tags: CSS
---

> 之前对于width属性下auto和100%的差异一直有误解,认为auto情况下容器的宽度是由内容撑开的,而100%是由父元素的宽度决定的.在写了一些样例测试后,对这两者的区别和实现有了新的认识


<!--more-->
#### width:100%
1. 宽度:
参照**定义了宽度属性的第一个父级元素**决定,通过**计算父级元素盒模型内的content宽度决定自身的content宽度**,**border、padding、margin的值不会被计算在内**,因此会有溢出父元素的情况出现
2. float情况:
在float情况下,区块脱离文档流,但是width仍然根据父级元素的width属性决定

#### width:auto
1. 宽度:
区块设置为width:auto之后，会根据**父元素的width，该元素本身的margin，该元素本身的border，该元素本身的padding**来一起决定该元素的width。 换句话说，_**该元素的width + padding * 2+border * 2 + margin * 2 = 父元素的width**_。
2. float情况:
在float情况下,区块脱离文档流,width:auto的根据父元素自适应效果失效,宽度由内容撑开