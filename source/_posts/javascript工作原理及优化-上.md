---
title: 每周博文|javascript 工作原理及优化
date: 2017-09-02 18:52:14
tags:
- 性能
- 翻译
categories:
- 周翻计划
description: 之前一直在说，要每周去读读好的外国网站的文章，最近是在看前端之巅每周推送的时候，选择了一篇文章，虽然是一篇软文，但是里面的内容确是实打实的，文章值得一读，产品也值得一试。文章有点长，能力也有点现，今天只完成部分，明天再来
---
<!-- more -->
# How JavaScript works: inside the V8 engine + 5 tips on how to write optimized code
<hr />
# 在 v8引擎下，js 是如何工作的以及5条书写高性能代码的建议
Couple of weeks ago we started a series aimed at digging deeper into JavaScript and how it actually works: we thought that by knowing the building blocks of JavaScript and how they come to play together you’ll be able to write better code and apps.

```
几周前，我们开始发表了一系列针对 js 进行更深层次挖掘和讲述 js 到底是怎么工作的文章。我们认为只有当你了解了 js 的构建块，并且了解这些构建块之间的关系的时候，你才有可能写出更好的代码和应用。

aimed at:针对
```
The first post of the series focused on providing an overview of the engine, the runtime and the call stack. This second post will be diving into the internal parts of Google’s V8 JavaScript engine. We’ll also provide a few quick tips on how to write better JavaScript code —best practices our development team at SessionStack follows when building the product.

```
初次发表的一系列文章的焦点是引擎的概况，运行和调用栈。
这是第二个系列，这个系列主要是更深入的去讲 v8引擎内部的一些东西。
我们当然也会涉及到怎么去写一段更好的 js 代码，我们这个团队在开发的过程中的一些最佳实践。

注：第一部分我还没看
```
## Overview

A JavaScript engine is a program or an interpreter which executes JavaScript code. A JavaScript engine can be implemented as a standard interpreter, or just-in-time compiler that compiles JavaScript to bytecode in some form.

This is a list of popular projects that are implementing a JavaScript engine:
  ● V8 — open source, developed by Google, written in C++
  ● Rhino — managed by the Mozilla Foundation, open source, developed entirely in Java
  ● SpiderMonkey — the first JavaScript engine, which back in the days powered Netscape Navigator, and today powers Firefox
  ● JavaScriptCore — open source, marketed as Nitro and developed by Apple for Safari
  ● KJS — KDE’s engine originally developed by Harri Porten for the KDE project’s Konqueror web browser
  ● Chakra (JScript9) — Internet Explorer
  ● Chakra (JavaScript) — Microsoft Edge
  ● Nashorn, open source as part of OpenJDK, written by Oracle Java Languages and Tool Group
  ● JerryScript — is a lightweight engine for the Internet of Things.

```
通常来讲 js 的引擎要么是一段程序，要么就是个解释器。
js 的解释器有两种，一种是标准的解释器，另一种是实时编译的解释器，这种实时编译的解释器会把代码翻译成字节码（bytecode）
下面的列表是目前比较流行的 js 引擎项目的列表，捡主要的说，有些听都没听过的就过了。
1-v8        谷歌          c++      开源
2-rhino     Mozilla      java     开源   
3-SpiderMonkey  firefox 接手网景 也是第一个 js 引擎
4-javascriptCore safari 苹果公司的开源项目 
5-chakra    IE(js9) Edge(js)

注：感觉这个表好有意思啊。
一会儿说说哈里波特(Harri Porten),一会儿有说说查克拉（chakra）^^          
```
## Why was the V8 Engine created?
The V8 Engine which is built by Google is open source and written in C++. This engine is used inside Google Chrome. Unlike the rest of the engines, however, V8 is also used for the popular Node.js runtime.

V8 was first designed to increase the performance of JavaScript execution inside web browsers. In order to obtain speed, V8 translates JavaScript code into more efficient machine code instead of using an interpreter. It compiles JavaScript code into machine code at execution by implementing a JIT (Just-In-Time) compiler like a lot of modern JavaScript engines do such as SpiderMonkey or Rhino (Mozilla). The main difference here is that V8 doesn’t produce bytecode or any intermediate code.

```
为什么要创建 v8引擎呢？
上面也说到过了，v8是谷歌家的开源项目，用在chrom 浏览器里，这个引擎和其他的引擎还不太一样，它也用在 NodeJs里。
v8是第一款以提升web 浏览器 js引擎 性能为目的进行设计的。为了提高速度，v8在解释 js 代码的时候，选择解析成更有效率的机器代码来代替解释器。这个解释为机器代码的过程是使用实时编译的的解释器，这一点和其他的一些引擎比如说 spiderMonkey 或是 rhino 是一样的。
真正提升的性能点是：在这个实时编译的过程中，v8不产生字节码或是其他的中间代码。

注，这就相当于同传了吧。
```
## V8 used to have two compilers
Before version 5.9 of V8 came out (released earlier this year), the engine used two compilers:
```
在5.9版本之前，v8引擎是有两个编译器的。
```
  ● full-codegen — a simple and very fast compiler that produced simple and relatively slow machine code.
  ```
  一个是full-codegen-相对来讲简单易用，速度快的,这个引擎产生简单但是相对来讲比较慢的机器代码
  ```
  ● Crankshaft — a more complex (Just-In-Time) optimizing compiler that produced highly-optimized code.
  ```
  另一个是 crankshaft-一个相对复杂点的实时编译产生性能更高的代码。
  ```
The V8 Engine also uses several threads internally:
```
除了有两个编译器之外呢，v8内部还有几个线程需要说一下：
```
  ● The main thread does what you would expect: fetch your code, compile it and then execute it
  ```
  主线程：会获取你的代码，编译，然后执行。
  ```
  ● There’s also a separate thread for compiling, so that the main thread can keep executing while the former is optimizing the code
  ```
  在主线程编译的时候呢，又分出了第二个线程，这个线程在主线程正常进行的前提下，进行代码优化。
  
  ```
  ● A Profiler thread that will tell the runtime on which methods we spend a lot of time so that Crankshaft can optimize them
  ```
  还有个用于分析统计的线程：用于告诉我们，在运行的过程中哪些方法比较费时间，这样 crankshaft 就去优化他们。
  ```
  ● A few threads to handle Garbage Collector sweeps
When first executing the JavaScript code, V8 leverages full-codegen which directly translates the parsed JavaScript into machine code without any transformation. This allows it to start executing machine code very fast. Note that V8 does not use intermediate bytecode representation this way removing the need for an interpreter.
```
还有一些线程用于操作处理垃圾回收器，以收集第一次运行 js 代码时产生的垃圾碎片。v8让 full-codegen这个简单快速的引擎去解析 js 代码为机器代码这部分是没变的，因为速度快。注意，v8不使用中间的字节码来表示自己不需要解释器了。（啥意思？）
```
When your code has run for some time, the profiler thread has gathered enough data to tell which method should be optimized.
Next, Crankshaft optimizations begin in another thread. It translates the JavaScript abstract syntax tree to a high-level static single-assignment (SSA) representation called Hydrogen and tries to optimize that Hydrogen graph. Most optimizations are done at this level.
```
当你的代码运行了一段时间之后，用于统计分析的的这个线程就有足够的分析数据，告诉你哪个方法应该优化。然后优化的这个引擎 crankshaft 开始在其他的线程中进行优化了。它会把 js 的抽象语法树翻译成一个更高级的，静态单一分配赋值的语法去表示，叫做 Hydrogen。（不太懂，太专业）然后试图去优化这个图标。在这个阶段绝大多数的优化就完成了。

```
Inlining
The first optimization is inlining as much code as possible in advance. Inlining is the process of replacing a call site (the line of code where the function is called) with the body of the called function. This simple step allows following optimizations to be more meaningful.
```
第一次优化是内联的。内联是一个过程，是当你在调用某个方法时，放到方法体中的，这个简单的步骤允许后面的优化更有意义（针对性？）
```


