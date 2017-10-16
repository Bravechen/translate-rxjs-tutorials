# 介绍

RxJS 是一个使用可观察量(observable)队列解决异步编程和基于事件编程的js库。
他提供了一个核心的类型`Observable`，和若干附属类型(`Observer`、`Schedulers`、`Subject`)以及一组的操作符(`map`,`filter`,`reduce`,`every`等等)，可以像操作集合一样处理异步事件流。

>Think of RxJS as Lodash for events.

为了较好的管理事件队列，响应式编程组合了观察者模式和迭代器模式，并且提供了操作集合的函数式编程方法。

RxJS提供了几个管理异步事件的核心概念:

- `Observable`: 可观察量，代表了一个由未来获取到的值或事件组成的集合。
- `Observer`:观察者，是一个集合，由监听Observable推送消息的一个或多个回调函数组成。
- `Subscription`:订阅过程，代表了Observable的执行过程，通常用来取消或者中断Observable的执行过程。
- `Operators`: 操作符是一些纯函数，用来采用函数式编程风格处理集合，比如：`map`,`filter`,`concat`,`flatMap`等等。
- `Subject`: Subject相当于事件触发器(EventEmitter)，是向多个Observer广播事件或推送值的唯一方法。
- `Schedulers`: 调度者集中了派发器(dispatcher)控制并发，允许我们在使用类似`setTimeout()`,`requestAnimationFrame`或其他方法时，协调计算。

## 第一个例子

通常你会像下面这样注册事件侦听器:

```js

var button = document.querySelector('button');
button.addEventListener('click', () => console.log('Clicked!'));

```

使用RxJS，你可以创建一个observable，处理相同的逻辑:

```js

var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
  .subscribe(() => console.log('Clicked!'));
```
## 纯函数

采用纯函数生产数据，使RxJS的能力很强，也可以减少代码出错的几率。

通常，一个不纯函数中的部分代码可能会扰乱状态，类似:

```js
var count = 0;
var button = document.querySelector('button');
button.addEventListener('click', () => console.log(`Clicked ${++count} times`));
```

使用RxJS，你可以将状态隔离起来:

```js

var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
  .scan(count => count + 1, 0)
  .subscribe(count => console.log(`Clicked ${count} times`));
```

例子中的`scan`操作符类似于数组的`reduce()`方法。他将一个值传递给回调函数，之后的返回值则会作为一个输入被传递给下一个时间点上的回调函数。

## 流

为了控制Observable实例中事件流，RxJS提供了各种各样的操作符。

以下的例子展示了使用纯js，实现至少间隔1秒发出一次点击事件的代码:

```js
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

使用RxJS，可以这样:

```js

var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
  .throttleTime(1000)
  .scan(count => count + 1, 0)
  .subscribe(count => console.log(`Clicked ${count} times`));
```

还有很多控制流的操作符，比如:`filter`,`delay`,`debounceTime`,`take`,`takeUntil`,`distinct`,`distinctUntilChanged`等等。

## 值

你可以改变Observable流中的值。

以下的例子，使用纯js实现了获取每次click事件时鼠标x坐标的值，并进行了计算:

```js

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
使用RxJS，你可以这样:

```js
var button = document.querySelector('button');
Rx.Observable.fromEvent(button, 'click')
  .throttleTime(1000)
  .map(event => event.clientX)
  .scan((count, clientX) => count + clientX, 0)
  .subscribe(count => console.log(count));
```

其他用来加工和生产值的操作符有:`pluck`,`pairwise`,`sample`等等。