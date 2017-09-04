---
title: 一周专题之es6-smybol
date: 2017-09-04 15:58:55
tags:
- es6
categories:
- 一周专题
description: 上个礼拜简单的过了下深入浅出ES6有关symbol的章节 过眼不过心 这周打算啃啃同时上周本来打算封装一个弹窗插件的，但是发现一个是自己的知识不够扎实，另一个是眼界不够，偏而不全。所以这个月也打算把javascript设计模式与开发实践》这本书攻下来同时学习设计心理学这一套书并且练习ps。本来是等前端之巅的文章的但是上周，发现翻译一篇文章还是很费力气的所以这周打算拆分。文章选择为：https://medium.com/@addyosmani/javascript-start-up-performance-69200f43b201 已有大大翻译，纯粹练习 http://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzIwNjQwMzUwMQ%3D%3D%26mid%3D2247484987%26idx%3D1%26sn%3D7f20da20bc6baed62ca8ff115209942b 愿下周前参悟透。落于实处，戒骄戒躁JS 
---

## 第六种原始类型  前面五种是什么?
在众多关于symbol的说明中，我最想知道，symbol他到底凭什么和string,boolean,number,null,undefined平起平坐。（这不！你就知道js的五壮士是谁了吧！）

## symbol作为原始值，我不服！单从创建上来讲他就不配！
```
var a;
```
就看这个例子，我们创建了变量a，现在我们想给a赋值。先来看看我们5位大哥是怎么做的。
```
a = "so easy";
a = true;
a = null;
a = undefined;
a = 99;
```
怎么样，神通吧。再看看symbol。说什么，得通过全局函数才能创建！苍天啊。不服！
（一阵喧闹……）
好，你初来乍到，先让让你。
```
var a = Symbol();
console.log(a);//【结果】Symbol()
```

我去你是个什么东西，逗我玩儿呢是不是！
对象爱看热闹啊，对象说要不我来试试。“来,你来做我的属性，老司机带你飞”
好嘛，symbol一句承让和对象结了对。

```
var a = Symbol();
console.log(a);
var obj = {};
obj[a] = "初来乍到";
console.log(obj);
console.log(obj[Symbol()]);//undefined
console.log(obj[a]);//初来乍到
```

好嘛！这可把对象吓了一跳。“大侄子，你这什么玩意!怎么这么不伦不类啊。你这函数不像函数，调用不像调用的。”

不好意思今天到这儿，本猿就懵了……