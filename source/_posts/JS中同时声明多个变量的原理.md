---
title: JS中同时声明多个变量的原理
date: 2016-05-08 10:28:27
tags: JavaScript
---
``` bash
function show(){
	var a=b=c=d=5;
}
show();
alert(a); //弹出a报错，显示undefined
```


#### Q：为什么在函数中，只有变量a被声明为局部变量？
因为赋值是**从右向左**结合；
*var a=b=c=d=5;*等价于*var a=(b=(c=(d=5)));*,  其中只有a被声明了，b,c和d都是自动解析为全局变量了。