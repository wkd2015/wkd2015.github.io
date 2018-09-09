---
title: history.back(-1)和history.go(-1)的区别
date: 2016-05-02 23:22:22
tags: HTML
---
history.back(-1)直接返回当前页的上一页，数据全部消息消失，是个新页面；
history.go(-1)也是返回当前页的上一页，不过表单里的数据全部还在。
``` bash
history.back(0) 刷新
history.back(1) 前进
history.back(-1) 后退
```