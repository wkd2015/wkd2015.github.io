---
title: jQuery事件绑定方法delegate()
date: 2016-05-26 21:22:33
tags: [JavaScript,jQuery]
---
### 定义
delegate()方法为指定的元素（属于被选元素的子元素）添加一个或多个事件处理程序，并规定当这些事件发生时运行的函数。
*使用delegate()方法的事件处理程序适用于未来或当前的元素（比如脚本创建的元素）*
<!--more-->
### 语法
``` bash
$(selector).delegate(childSelector,event,data,function);
```
| 参数        | 描述   |
| ----   | --------------------  |
| childSelector     | 必需。规定要附加事件处理程序的一个或多个子元素 |
| event        |   必需。规定附加到元素的一个或多个事件。由空格分隔多个事件值。必须是有效的事件。   |
| data        |    可选。规定传递到函数的额外数据。    |
| function    |   必需。规定当事件发生时运行的函数。   |



### 其他事件绑定方法
1. bind(type,[data],fn) 为每个匹配元素的特定事件绑定事件处理函数
``` bash
$("a").bind("click",function(){alert("ok");});
```
2. live(type,[data],fn) 给所有匹配的元素附加一个事件处理函数，即使这个元素是以后再添加进来的
``` bash
$("a").live("click",function(){alert("ok");});
```
3. on(events,[selector],[data],fn) 在选择元素上绑定一个或多个事件的事件处理函数
``` bash
$("table").on("click", "td", function(){alert("hi");});
```
#### 差别
> * .bind()是直接绑定在元素上
> * .live()则是通过冒泡的方式来绑定到元素上的。更适合列表类型的，绑定到document DOM节点上。和.bind()的优势是支持动态数据。
> * .delegate()则是更精确的小范围使用事件代理，性能优于.live()
> * .on()则是最新的1.9版本整合了之前的三种方式的新事件绑定机制