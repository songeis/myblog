---
title: 每周博文|javascript 工作原理及优化-下
date: 2017-09-03 19:14:48
tags:
- 性能
- 翻译
categories:
- 周翻计划
description: 之前一直在说，要每周去读读好的外国网站的文章，最近是在看前端之巅每周推送的时候，选择了一篇文章，虽然是一篇软文，但是里面的内容确是实打实的，文章值得一读，产品也值得一试。文章有点长，能力也有点现。今天就算是完了吧，感觉有点云里雾里的。还得好好消化。
---

## Hidden class
JavaScript is a prototype-based language: there are no classes and objects are created using a cloning process. JavaScript is also a dynamic programming language which means that properties can be easily added or removed from an object after its instantiation.

```
js 是个基于原型的语言，所谓基于原型的语言是说这里面没有类的概念并且对象的创建这个过程是通过克隆完成的。同时，js 也是一个动态的编程语言，所谓动态表示，属性可以在对象实例化以后也可以轻易的增加和删除。
```
Most JavaScript interpreters use dictionary-like structures (hash function based) to store the location of object property values in the memory. This structure makes retrieving the value of a property in JavaScript more computationally expensive than it would be in a non-dynamic programming language like Java or C#. In Java, all of the object properties are determined by a fixed object layout before compilation and cannot be dynamically added or removed at runtime (well, C# has the dynamic type which is another topic). As a result, the values of properties (or pointers to those properties) can be stored as a continuous buffer in the memory with a fixed-offset between each. The length of an offset can easily be determined based on the property type, whereas this is not possible in JavaScript where a property type can change during runtime.
```
许多 js 的解释器使用类似字典的结构（基于 hash 函数）在内存中存储中对象属性的位置。这种结构使得在 js 中检索属性值的这个过程比起一些非动态的语言比如说 java 或是 C#这样要花费更多的计算。
在 java 中，所有对象的属性在计算之前是固定在一个对象结构里的，并且在运行的过程中不能动态的添加或删除。（而 C#其实也是有动态类型的，不过那是另一个话题。）作为结果来讲，属性值（或者说指向这些属性的指针）是可以存储成一个连续的 buffer 在内存中，每个 buffer 之间都有固定的偏移量。偏移量的值是可以很轻松的根据属性的类型决定的。
但是，这在 js 中是不太可能的，因为js 在运行时属性的类型是可以动态的改变的。
```
Since using dictionaries to find the location of object properties in the memory is very inefficient, V8 uses a different method instead: hidden classes. Hidden classes work similarly to the fixed object layouts (classes) used in languages like Java, except they are created at runtime. Now, let’s see what they actually look like:
```
在内存中使用字典去查找对象属性的位置是非常低效的，v8使用一个不同的方法来代替，这个不同的方法就是hidden classes(隐藏类) hidden classes 工作原理和 java 中的固定对象布局很像，但 hidden classes 是在运行时创建的。
现在让我们来看看到底什么是 hidden class ，来看下面一段代码：
```
```
function Point(x, y) {
    this.x = x;
    this.y = y;
}
var p1 = new Point(1, 2);
```
Once the “new Point(1, 2)” invocation happens, V8 will create a hidden class called “C0”.
```
一旦 new Point 实例化的调用发生，v8就会创建一个叫 C0的隐藏类。
```
No properties have been defined for Point yet, so “C0” is empty.
Once the first statement “this.x = x” is executed (inside the “Point” function), V8 will create a second hidden class called “C1” that is based on “C0”. “C1” describes the location in the memory (relative to the object pointer) where the property x can be found. In this case, “x” is stored at offset 0, which means that when viewing a point object in the memory as a continuous buffer, the first offset will correspond to property “x”. V8 will also update “C0” with a “class transition” which states that if a property “x” is added to a point object, the hidden class should switch from “C0” to “C1”. The hidden class for the point object below is now “C1”.
```
Point 函数里还没有属性定义，所以 C0是空的。
一旦 this.x = x 在函数 Point 内部执行，v8就会创建第二个隐藏类叫 C1。C1是基于 C0创建的。C!描述了x 在内存中的位置（相对于对象指针），在这个李志忠 x 被存储在0这个位置上，这就意味着当在内存中查看 point 对象的时候可以，把 point 对象当做一个 连续的buffer对象，第一个偏移量就是属性 x。
v8当属性 x 添加到一个 point 对象的时候，v8也会更新 C0，而hidden class 隐藏类也会从 C0切换到 C1.这个时候Point object 下的隐藏类就应该是C1。
```

Every time a new property is added to an object, the old hidden class is updated with a transition path to the new hidden class. Hidden class transitions are important because they allow hidden classes to be shared among objects that are created the same way. If two objects share a hidden class and the same property is added to both of them, transitions will ensure that both objects receive the same new hidden class and all the optimized code that comes with it.
```
每次新的属性添加到一个对象的时候，原先的Hidden Class(隐藏类)就会更新路径到新的隐藏类。隐藏类的之间的转变是很重要的，因为他们允许隐藏类在以同样方式常见的对象之间共享。如果两个对象共享一个隐藏类并且同样的属性都在这两个对象中添加了，那么转换就会保证这两个对象收到同样的隐藏类并且所有的优化的代码都是如此实现的。
```
This process is repeated when the statement “this.y = y” is executed (again, inside the Point function, after the “this.x = x” statement).
A new hidden class called “C2” is created, a class transition is added to “C1” stating that if a property “y” is added to a Point object (that already contains property “x”) then the hidden class should change to “C2”, and the point object’s hidden class is updated to “C2”.
```
上面说的过程在this.y=y 执行的时候再次在函数 Point内执行执行，就会产生新的Hidden class（隐藏类）叫 C2,隐藏类的切换也从 C1变为了 C2
```
Hidden class transitions are dependent on the order in which properties are added to an object. Take a look at the code snippet below:
```
隐藏类切换取决于属性添加到对象的顺序，看看下面的代码
```
```
function Point(x, y) {
    this.x = x;
    this.y = y;
}
var p1 = new Point(1, 2);
p1.a = 5;
p1.b = 6;
var p2 = new Point(3, 4);
p2.b = 7;
p2.a = 8;
```
Now, you would assume that for both p1 and p2 the same hidden classes and transitions would be used. Well, not really. For “p1”, first the property “a” will be added and then the property “b”. For “p2”, however, first “b” is being assigned, followed by “a”. Thus, “p1” and “p2” end up with different hidden classes as a result of the different transition paths. In such cases, it’s much better to initialize dynamic properties in the same order so that the hidden classes can be reused.
```
现在你应该假定p1和 p2都使用的同样的hidden Class 和 转换。但还不准确，对p1，先添加属性 a然后再添加的属性 b,但是p2先添加的 b 然后再添加的 a。因此，p1和p2实际上不是用的同一个隐藏类和转换。
在这种情况下，最好使用同样的顺序添加属性，以便隐藏类可以复用
```
## Inline caching
V8 takes advantage of another technique for optimizing dynamically typed languages called inline caching. Inline caching relies on the observation that repeated calls to the same method tend to occur on the same type of object. An in-depth explanation of inline caching can be found here.

We’re going to touch upon the general concept of inline caching (in case you don’t have the time to go through the in-depth explanation above).
```
v8优化动态类型的语言还有另外一项技术，叫 inline caching(内联缓存)
内联缓存依赖观察，一种在同一种类型的对象中对重复的调用同一个函数的换观察。
我们还会做出一种更深入的解释
我们试图去涉及一些关于 inline caching更寻常一点的概念。
（事实上，你不会有时间去深入的去解释上面的问题。）
```
So how does it work? V8 maintains a cache of the type of objects that were passed as a parameter in recent method calls and uses this information to make an assumption about the type of object that will be passed as a parameter in the future. If V8 is able to make a good assumption about the type of object that will be passed to a method, it can bypass the process of figuring out how to access the object’s properties, and instead, use the stored information from previous lookups to the object’s hidden class.
```
所以 inline caching 到底是怎么工作的的？
v8 有个对象缓存作为参数传入到最近调用的函数中，并且这些信息（this information）会假设对象会在未来再次作为参数传递。如果 v8可以对函数接收的一个对象类型的参数做出正确的假设，那么就会绕开并代替这个如何进行对象访问的进程，使用以前查找时在 hidden class 里的存储信息
```
So how are the concepts of hidden classes and inline caching related? Whenever a method is called on a specific object, the V8 engine has to perform a lookup to the hidden class of that object in order to determine the offset for accessing a specific property. After two successful calls of the same method to the same hidden class, V8 omits the hidden class lookup and simply adds the offset of the property to the object pointer itself. For all future calls of that method, the V8 engine assumes that the hidden class hasn’t changed, and jumps directly into the memory address for a specific property using the offsets stored from previous lookups. This greatly increases execution speed.
```
所以，hidden class 和 inline caching 到底是怎么关联的呢？
当一个方法被一个特定对象调用，v8引擎就会执行该对象的隐藏类的查找，为了找到一个特殊属性的定位信息。在两个拥有同一个隐藏类的方法调用成功后，v8就会删掉这个 hidden class（隐藏类）并且简单的添加这个属性的定位信息到对象 pointer本身。为了以后每次调用，v8假设隐藏类不会再变化，并且直接进入之前查询过得特殊属性的定位信息在内存中的地址，这大大的提升了执行熟读。

```
Inline caching is also the reason why it’s so important that objects of the same type share hidden classes. If you create two objects of the same type and with different hidden classes (as we did in the example earlier), V8 won’t be able to use inline caching because even though the two objects are of the same type, their corresponding hidden classes assign different offsets to their properties.
```
inline caching 也是很重要的一个原因，同一个类型的对象可以共享一个 hidden class 的原因。如果你创建两个相同类型的对象并且使用不同的 hidden class。就像我们之前说的那个例子。v8是不能使用inline caching 的，因为尽管这两个对象都是同一个类型，但是他们记录的属性的偏移量是不一致的（属性 abba 的例子）
```
## Compilation to machine code
Once the Hydrogen graph is optimized, Crankshaft lowers it to a lower-level representation called Lithium. Most of the Lithium implementation is architecture-specific. Register allocation happens at this level.
```
一旦 Hydrogn graph 完成优化，crankshaft引擎就会把 Hydrogen graph 的优先级降低，这叫做 lithium。许多 lithium 都是基于特定的体系和结构的。注册分配就发生在这个阶段。
```
In the end, Lithium is compiled into machine code. Then something else happens called OSR: on-stack replacement. Before we started compiling and optimizing an obviously long-running method, we were likely running it. V8 is not going to forget what it just slowly executed to start again with the optimized version. 
```
最后，lithium 是会编译成机器代码的。然后发生的事情叫 OSR，栈的替换。
在我们开始编译和优化一段长期运行的方法的时候，我们很可能还在运行着他。v8不会忘了什么使得运行变得缓慢，并且再次优化。
```
Instead, it will transform all the context we have (stack, registers) so that we can switch to the optimized version in the middle of the execution. This is a very complex task, having in mind that among other optimizations, V8 has inlined the code initially. V8 is not the only engine capable of doing it.
```
作为代替，它会改变我们所有的上下文这就使得我们可以在运行中切换优化的版本。这是一项非常复杂的任务，在其他优化中间，v8还要内联进行代码初始化，v8不是唯一一个这么干的引擎
```
There are safeguards called deoptimization to make the opposite transformation and revert back to the non-optimized code in case an assumption the engine made doesn’t hold true anymore.
```
deoptimization 是个保障。他保证了，当引擎的假设失败的时候他可以反向的编译并回退到没有优化的代码。
```
## Garbage collection
For garbage collection, V8 uses a traditional generational approach of mark-and-sweep to clean the old generation. The marking phase is supposed to stop the JavaScript execution. In order to control GC costs and make the execution more stable, V8 uses incremental marking: instead of walking the whole heap, trying to mark every possible object, it only walk part of the heap, then resumes normal execution. The next GC stop will continue from where the previous heap walk has stopped. This allows for very short pauses during the normal execution. As mentioned before, the sweep phase is handled by separate threads.
```
对于垃圾回收来说，v8使用了一种传统方式清除旧的产生垃圾。标记阶段应该停止 js 的执行。为了控制 GC 成本和更稳健的运行，v8使用增量标记代替：（walking the whole heap？）尝试标记每个可能的对象，他只在堆的一部分中执行，然后恢复正常的工作。然后下次 GC 停止就在之前的 GC 停止过的地方再次停下。这允许在正常的运行过程中有短暂的暂停。然后就想前文所说的那样，扫描的阶段由单独的线程执行。
```
## Ignition and TurboFan
With the release of V8 5.9 earlier in 2017, a new execution pipeline was introduced. This new pipeline achieves even bigger performance improvements and significant memory savings in real-world JavaScript applications.
The new execution pipeline is built on top of Ignition, V8’s interpreter, and TurboFan, V8’s newest optimizing compiler.
```
随着2017年的5.9的版本而出的一个新的管道。这个新的管道使得 js 应用的性能和节省内存都得到了很大的提升。这个管道建立在 Ignition（点火）的基础上，v8的编译器和TurboFan最新的性能化的编译器。
```
You can check out the blog post from the V8 team about the topic here.
Since version 5.9 of V8 came out, full-codegen and Crankshaft (the technologies that have served V8 since 2010) have no longer been used by V8 for JavaScript execution as the V8 team has struggled to keep pace with the new JavaScript language features and the optimizations needed for these features.
This means that overall V8 will have much simpler and more maintainable architecture going forward.
```
详情你可以查看 v8的官方博客。
到5.9版本出来，full-codegen和 crankshaft 这两个引擎渐渐将不再使用，这意味着，v8将会变得更简单更高效
```
These improvements are just the start. The new Ignition and TurboFan pipeline pave the way for further optimizations that will boost JavaScript performance and shrink V8’s footprint in both Chrome and Node.js in the coming years.
Finally, here are some tips and tricks on how to write well-optimized, better JavaScript. You can easily derive these from the content above, however, here’s a summary for your convenience:
```
这些变化也只是个起步阶段，新的 Ignition 和 TurboFan 是v8团队未来几年的发展重点。最后有一些建议，关于如何写出高效的，优化的 js 代码。
```
## How to write optimized JavaScript
  1. Order of object properties: always instantiate your object properties in the same order so that hidden classes, and subsequently optimized code, can be shared.
  ```
  总是以相同的顺序添加你的对象属性，以便 hidden class 和随后的优化能够复用。
  ```
  2. Dynamic properties: adding properties to an object after instantiation will force a hidden class change and slow down any methods that were optimized for the previous hidden class. Instead, assign all of an object’s properties in its constructor.
  ```
  动态属性：在实例化后添加属性到对象将会强制修改隐藏类，并且降低隐藏类的优化。代替的分配所有在对象的所有的属性
  
  ```
  3. Methods: code that executes the same method repeatedly will run faster than code that executes many different methods only once (due to inline caching).
  ```
  方法：重复执行相同的代码会比执行一次的代码要快许多。
  ```
  4. Arrays: avoid sparse arrays where keys are not incremental numbers. Sparse arrays which don’t have every element inside them are a hash table. Elements in such arrays are more expensive to access. Also, try to avoid pre-allocating large arrays. It’s better to grow as you go. Finally, don’t delete elements in arrays. It makes the keys sparse.
  ```
  数组：避免使用非递增索引的数组，这种叫稀疏数组，稀疏数组的查找代价更大。也避免预先定义一个大数组，动态的像数组追加的方式更好。也不要删除数组元素，这样会产生稀疏数组。
  ```
  5. Tagged values: V8 represents objects and numbers with 32 bits. It uses a bit to know if it is an object (flag = 1) or an integer (flag = 0) called SMI (SMall Integer) because of its 31 bits. Then, if a numeric value is bigger than 31 bits, V8 will box the number, turning it into a double and creating a new object to put the number inside. Try to use 31 bit signed numbers whenever possible to avoid the expensive boxing operation into a JS object.
  ```
  标记值：v8表现32位的对象和数字，判断是否是对象就看flag=1还是 flag=0，flag=0叫做 SMI（small Integer）因为他只有31位。如果一个数字类型的值大于31位了，v8就会把这个数字转换成浮点型并且创建一个新的对象把浮点数装进去。
  尝试使用31位标记数字，才能避免这个昂贵的转换。
  ```
  
  后面是他们团队自己的产品，我就不翻译了，感兴趣的话可以去用个试试。

附上原文链接：https://blog.sessionstack.com/how-javascript-works-inside-the-v8-engine-5-tips-on-how-to-write-optimized-code-ac089e62b12e



