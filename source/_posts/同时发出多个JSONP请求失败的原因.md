---
title: 同时发出多个JSONP请求失败的原因
date: 2018-03-31 23:35:56
tags: [JavaScript,JSONP]
---


> 在用jsonp跨域请求多个数据时，出现了一些错误，结合遇到的问题学习了下jsonp的逻辑以及解决方法。

<!-- more -->

### 概念
1. 跨域  
域指的是**协议+域名+端口**，三者只要有任何一个不同就表示不在同一个域里面，而访问不在一个域里的数据的行为，就叫做跨域。
2. JSONP  
JSONP是一种常用的跨域方法，只支持JSON格式的数据。基本原理是利用HTML的&lt;script&gt;标签天生就可以跨域的特点，用以加载另一个域的JSON数据，加载完成之后会自动运行一个回调函数来通知调用者。这个过程是需要另一个域的服务端支持的。而且只支持GET请求。
### 问题
在对需要同时发出的多个跨域请求进行处理时，出现了只有一个请求会成功调取回调函数的情况:  
**代码如下:**
```
if (url.indexOf('?') === -1) {
    url += '?cb=responseHandler';
} else {
    url += '&cb=responseHandler';
}
let script = document.createElement('script');
eval(`responseHandler = function (json) { 
    try {
        resolve(json);
    } finally {
        script.parentNode.removeChild(script);
        window['responseHandler'] = null;
    }
}`);
script.setAttribute('src', url)
document.body.appendChild(script);
```
**运行结果如下:**  
![](https://raw.githubusercontent.com/wkd2015/Pic/master/blog/jsonp_1.jpg '描述')
> 在这里共有12次请求，**请求全部成功发出并返回，但只有最后一次请求的回调函数生效**，而其余的全部失败。

### 分析
1. 在上述代码中，其实是往windows全局环境中添加了一个方法responseHandler以处理请求的回调函数，从而接收数据。而这12次操作，则是由后一个接替覆盖了前一个函数表达式的赋值操作，最后仅剩下了最后一个jsonp请求的处理。因此，**实际上多次请求调用的都是最后一次jsonp的回调处理函数**。
2. 而eval中的函数代码执行的作用域为当前script标签所在的作用域，里面调用的函数也是最后一次生成的script标签，所以**报错的原因是重复执行了多次移除操作，其中只有一次是成功的**。

### 解决方法
给每一个JSONP请求的回调函数设定唯一的函数名，从而避免冲突，经过测试可以成功解决:

```
function jsonp(options) {
    let key = +new Date();
    let _url = options.url;
    if (_url.indexOf('?') === -1) {
        _url += '?cb=responseHandler' + key;
    } else {
        _url += '&cb=responseHandler' + key;
    }
    let script = document.createElement('script');
    eval(`responseHandler${key} = function (json) { 
        try {
            options.success(json)
        } finally {
            script.parentNode.removeChild(script);
            window['responseHandler${key}'] = null;
        }
    }`);
    script.setAttribute('src', _url);
    document.body.appendChild(script);
}
```
### 扩展：JSONP的失败检测
1. 定时器
```
//other code
eval(`responseHandler${key} = function (json) { 
    try {
        options.success(json)
    } finally {
        script.parentNode.removeChild(script);
        clearTimeout(_o.timer);
        window['responseHandler${key}'] = null;
    }
}`);
if(options.time) {
    _o.timer = setTimeout(function () {
        window['responseHandler' + key] = null;
        script.parentNode.removeChild(script);
        options.fail && options.fail({msg: "timeout"});
    }, options.time);
}
//other code
```
2. onerror
```
//other code 
script.onerror = function (e) {
    window['responseHandler' + key] = null;
    script.parentNode.removeChild(script);
    options.fail && options.fail({msg: "timeout"});
}
//other code
```

> onerror是HTML5新增方法，部分老旧的浏览器不支持。