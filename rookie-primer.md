_
本篇对应于官方的介绍篇，因英文介绍与gitbook文件名冲突，所以改了一下_


RxJS是一个通过使用可观察序列来构建**异步**和**基于事**件的程序的库。它提供了一个核心类型:**Observable**、卫星类型(大概是这些类型均围绕于Observable，也就是Observable是根基，而这些是辅助类型):**Observer**、**Schedulers**、**Subjects**)和操作符-衍生自一些数组方法，使得我们可以把异步事件以集合的方式进行处理。
>把RxJS当做一个针对事件的Lodash(一个JS库)。

ReactiveX将观察者模式与迭代器模式和使用集合的函数式编程组合在一起，来满足这种管理事件序列的理想方式

RxJS中解决异步事件管理的基本概念如下：
* Observable可观察对象：表示一个可调用的未来值或者事件的集合。
* Observer观察者：一个回调函数集合,它知道怎样去监听被Observable发送的值
* Subscription订阅： 表示一个可观察对象的执行，主要用于取消执行。
* Operators操作符： 纯粹的函数，使得以函数编程的方式处理集合比如:map,filter,contact,flatmap。
* Subject(主题)：等同于一个事件驱动器，是将一个值或者事件广播到多个观察者的唯一途径。
* Schedulers(调度者)： 用来控制并发，当计算发生的时候允许我们协调，比如setTimeout,requestAnimationFrame。


# 第一个例子
通常你这样注册事件监听：

```
var button = document.querySelector('button');
button.addEventListener('click', () => console.log('Clicked!'));
```
使用RxJS创建一个可观察对象：
```
var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
.subscribe(() => console.log('Clicked!'));
```

#Purity
RxJX能够使用纯函数的方式生产值的能力使得它强大无比。这意味着你的代码不再那么频繁的出现错误提示。

通常情况下你会创造一个非纯粹的函数，然后你的代码的其他部分可能搞乱你的程序状态。
```
var count = 0;
var button = document.querySelector('button');
button.addEventListener('click', () => console.log(`Clicked ${++count} times`));
```
使用RxJS来隔离你的状态

```
var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
.scan(count => count + 1, 0)
.subscribe(count => console.log(`Clicked ${count} times`));

```
scan操作符和数组中reduce方法的类似， 它需要一个传递给回调函数的参数值。 回调函数的返回值将成为下一次回调函数运行时要传递的下一个参数值。
#Flow 流

RxJS有着众多的操作符，可以帮助您控制事件如何流入可观察对象observables。


每秒最多只能点击一次的实现，使用纯JavaScript：


```
var count = 0;
var rate = 1000;
var lastClick = Date.now() - rate;
var button = document.querySelector('button');
button.addEventListener('click', () => {
if (Date.now() - lastClick >= rate) {
console.log(`Clicked ${++count} times`);
lastClick = Date.now();
}
});

```
使用RxJS


```
var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
.throttleTime(1000)
.scan(count => count + 1, 0)
.subscribe(count => console.log(`Clicked ${count} times`));
```

其他的流操作符是filter, delay, debounceTime, take, takeUntil, distinct, distinctUntilChanged 等等。
#Values值
你可以通过可观察对象来转化值

下面的程序可以在每次点击鼠标时获取X坐标位置

纯的JS实现



```
var count = 0;
var rate = 1000;
var lastClick = Date.now() - rate;
var button = document.querySelector('button');
button.addEventListener('click', (event) => {
  if (Date.now() - lastClick >= rate) {
    count += event.clientX;
    console.log(count)
    lastClick = Date.now();
  }
});
```

RxJS实现

```
var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
.throttleTime(1000)
.map(event => event.clientX)
.scan((count, clientX) => count + clientX, 0)
.subscribe(count => console.log(count));
```
其他的值生产者还有 pluck, pairwise, sample 等等.













