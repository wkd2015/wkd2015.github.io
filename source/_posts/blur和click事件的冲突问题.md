---
title: blur和click事件的冲突问题
date: 2016-08-10 23:26:02
tags: [JavaScript,DOM]
---

> 在input元素上同时绑定blur和click事件时

##### 1.问题

> blur事件和click事件各自的触发条件会导致本次绑定事件的冲突发生

<!--more-->


![输入框元素](https://raw.githubusercontent.com/wkd2015/pics/master/list.jpg)

在上图的每个input元素上绑定了blur和click事件处理程序.

1. input-click:当input被点击时,输入框获得焦点,下方出现对应操作按钮
![输入框元素](https://raw.githubusercontent.com/wkd2015/pics/master/after.jpg)
2. input-blur: 当input失去焦点时,input的值恢复到修改前的内容
![输入框元素](https://raw.githubusercontent.com/wkd2015/pics/master/before.jpg)
3. 按钮各自绑定了click事件

问题正出在按钮的click动作上,当按钮click时,值恢复到修改之前,导致传递出去的值也是修改前的.

##### 2.原因

click时间的触发条件是**一次点击**,一次点击包括:**鼠标点下,鼠标松开**,在点下时,input就已经失去焦点,导致blur事件触发.

##### 3.解决

各按钮绑定事件换为mousedown,而且测试发现**mousedown事件在有输入框获得焦点的情况下，它的默认动作会触发输入框的失去焦点事件**。如果清除默认动作(preventDefault())，那么输入框就不会失去焦点。