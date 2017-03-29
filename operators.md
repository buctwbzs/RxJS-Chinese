**由于操作符数量过多，加上用法介绍，会导致译文过于冗杂，而且官方文档的点击需要的类型层层的选择也很麻烦。为了查阅方便，将会在一个单独的RxJS类模块进行整理介绍，类上定义的每一个操作符都会单独介绍并附上源码，知其然更要知其所以然。如需查阅，请直接前往类模块。此文仅涵盖操作符的基本介绍，以及一个操作符是怎样产生的。**

# Operators操作符
RxJS如此强大的原因正是源自于它的操作符，即便可观察对象扮演着根基的角色。 操作符使得复杂的异步代码轻松的以声明的方式容易地组成。

#### 什么是操作符？
操作符是可观察对象上定义的方法，例如.map(...),.filter(...),.merge(...)，等等。当他们被调用，并不会去改变当前存在的可观察对象实例。相反，他们返回一个新的可观察对象，而且新返回的subscription订阅对象逻辑上基于调用他们的我观察对象。

>每一个操作符都是基于当前可观察对象创建一个新的可观察对象的函数。这是一个单纯无害的操作:之前的可观察对象仍然保持不变。

一个操作符本质上是一个将某个可观察对象作为输入然后输出另一个可观察对象的纯函数。对输出的新的可观察对象进行订阅的同时也会订阅作为输入的那个可观察对象。下面的例子，我们创建一个自定义的运算符函数，将从作为输入的可观察对象的每个值乘以10.
```js
function multiplyByTen(input) {
var output =Rx.Observable.create(function(observer){
input.subscribe({
next: (v) => observer.next(10*v),
error: (err) => observer.error(err),
complete: () => observer.complete()
});
});
return output;
}
var input = Rx.Observable.from([1,2,3,4]);
var output = multiplyByTen(input);
output.subscribe(x=>console.log(x));
```
输出如下:
```
10
20
30
40
```
注意一下我们对output的订阅导致了作为输入的可观察对象也被订阅了。我们称这种现象为"操作符的订阅链"。

#### 实例操作符(instance operator)VS静态操作符(static operator)

**什么是实例操作符？**通常，当引用运算符，我们假设是来自可观察对象实例的运算符。作为示例，如果multiplyByTen一个官方库中的操作符，那它大概是下面的样子:

```js
Rx.Observable.prototype.multiplyByTen = function(){
var input=this;
return Rx.Observable.create(function subscribe(observer){
input.subscribe({
next: (v) => observer.next(10*v),
error: (err) => observer.error(err),
complete: () => observer.complete()
});
});
}
```
译者注:考虑到可能会有些刚接触的人不太理解。上面的代码是这样的，操作符定义在可观察对象的原型对象上，如果某个可观察对象调用这个方法，input首先取得作为输入的该可观察对象的引用(后面会在闭包里用到)，然后该方法返回一个由create方法创建的可观察对象，也就是作为输出的可观察对象。而input.subscribe({...})是对input进行订阅。


>实例操作符是使用this关键词来推演出"输入可观察对象"是神马东东

注意input并不是作为函数的参数，而是作为this所指代的那个对象。下面是我们如何使用这个操作符:

```js
var observable = Rx.Observable.from([1,2,3,4]).multiplyByTen();
observable.subscribe(x => console.log(x));
```

**什么是静态操作符？**不同于实力操作符，静态操作符是直接定义在类上的。一个静态操作符并不在其内部使用this，而是完全依赖于它的参数。
> 静态操作符是定义在类上的函数，通常被用于从头重新创建一个可观察对象。

最常规的静态操作符是被称作构造操作符(Creation Operators，原谅我如此翻译，总是隐隐觉得很合适~)。不同于将一个输入的可观察对象转换成一个输出的可观察对象，它们仅接收一个非可观察对象作为参数，比如一个数字，然后构造出一个可观察对象。

一个静态操作符的典型例子是interval函数。它接收一个数字(而不是一个可观察对象)作为输入的参数，然后到一个可观察对象作为输出。
```js
var observable = Rx.Observable.intervable(1000)//1000毫秒
```
另一个构造操作符是create,我们在之前的例子中曾大量使用，一些合并操作符可能是静态的，例如:merge,combineLatest,concat,等等。将这些操作符声明为静态的是有意义的，因为他们接收多个可观察对象作为参数，而不是一个，示例如下:

```js
var observable1 = Rx.Observable.interval(1000);
var observable2 = Rx.Observable.interval(400);
var merged = Rx.Observable.merge(observable1, observable2);
```
#### Marble diagrams弹珠图标
使用文字来形容操作符到底是怎样得工作模式终究还是有那么点力不从心。许多的操作符是和时间线密不可分的，在实际中他们可能需要对值进行延迟、采样、节流、或者抖动的发射。使用图表通常能够对他们的过程进行更好的表述。弹珠图标能够囊括作为输入的可观察对象、操作符、及其参数、作为输出的可观察对象来对操作符的工作方式进行生动的表示。

> 在一个弹珠图标中，时间流向右侧，操作符描述了值(弹珠)在可观察对象执行时被怎样的发射。

下图你可以看到对整个弹珠图表流程的分析。![](/assets/1.png)

贯穿此后的文档，我们将会广泛的使用大量示例图标来解释操作的工作过程。他们在其它的内容的表述上也是如此的的有用,例如在一个白板或者我们的单元测试中。




























