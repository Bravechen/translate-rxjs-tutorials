# Observable 可观察量

可观察量是一种能惰性推送的集合，他可以包含多个值。下面的表格对比了推送和拉取2种方式：

| 类型 | 单值 | 多值 |
|--|--|--|
|拉取|Function|Iterator|
|推送|Promise|Observable|
 
举例来说，下列Observable 在被订阅之后会立即推送(同步)值`1,2,3`，而值`4`将会在1秒之后被推送到订阅者，之后这个流进入完成状态:
 
```js
var obserable = Rx.Observable.create(function(observer){   //注意参数的命名
 observer.next(1);
 observer.next(2);
 observer.next(3);
 
 setTimeout(function(){
     observer.next(4);
     observer.complete();
 },1000);
 
});
```
 
为了调用以上的Observable并输出其中的值，需要订阅这个流:

```js
console.log('just before subscribe');
observable.subscribe({
 next:x => console.log('got value:',x),
 error:err => console.log('something wrong occurred: ' + err),
 complete:() => console.log('done')
});
console.log('just after subscribe');

```
执行之后会输出:

```js
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```
 
## pull vs push 拉取 vs 推送
 
拉取和推送是数据生产者和数据使用者之间进行数据交换的2种形式。
 
### 什么是拉取？
 
在拉取系统中，数据使用者决定何时从数据生产者那里接收数据。数据生产者自身并不知道数据何时会被传递给使用者。

每一个js函数都是一个拉取系统。函数是一个数据生产者，通过在别处被调用，他返回的单一值将会被投入计算。

ES2015又引入了新的拉取系统，generator和迭代器(`function*`)。调用`iterator.next()`来消费从迭代器(数据生产者)中拉取到的数据。

| | Producer | Consumer |
|--|--|--|
|Pull|被动的：被请求的时候返回数据 |主动的：决定数据何时被请求 |
|Push|主动的：按照自己的步骤返回数据 |被动的：接收的数据后作出反应 |

### 什么是推送？

在推送系统中，数据生产者决定何时把数据推送给使用者，使用者自己并不知道何时能够接收到数据。

Promise是当今js中最常见的一种推送系统。和函数不同，Promise(数据生产者)精确的控制时机，将数据“推送”给业已注册的回调函数(数据使用者)。

RxJS引入了Observables，一个新的JS推送系统。

一个Observable是一个包含多个值的“数据生产者”，它会将数据推送给Observer(数据使用者)。

A Function is a lazily evaluated computation that synchronously returns a single value on invocation.
A generator is a lazily evaluated computation that synchronously returns zero to (potentially) infinite values on iteration.
A Promise is a computation that may (or may not) eventually return a single value.
An Observable is a lazily evaluated computation that can synchronously or asynchronously return zero to (potentially) infinite values from the time it's invoked onwards.

- `Function`是一种惰性计算方式，当他被调用时会同步的返回一个值。
- `generator`是一种惰性计算方式，会在迭代中同步的返回0到无限个（可能的话）返回值。
- `Promise`使用一种处理方法，最终可能会（或可能不会）返回一个值。
- `Observable`是一种惰性处理方法，当被调用的时候，可以同步也可以异步的返回0到无限个（可能的话）返回值。
 
### 理解Observable

有别于流行的说法，Observable既不像多个`EventEmitter`，也不像一种能返回多个值的`Promise`。

在一些场合，比如广播消息时，Observer看起来很像`EventEmitter`，在RxJS中，被称为`Subject`。而在大多数情况下，Observable并不像`EventEmitter`。

> Observables 像一群没有参数的函数，形成了多个值的集合。

考虑以下情况：

```js
function foo() {
  console.log('Hello');
  return 42;
}

var x = foo.call(); // same as foo()
console.log(x);
var y = foo.call(); // same as foo()
console.log(y);
```
输出是:

```
"Hello"
42
"Hello"
42
```

用Observables重写上面的行为是：

```js
var foo = Rx.Observable.create(function(observer){
    console.log('Hello');
    observer.next(42);
});

foo.subscribe(function(x){
    console.log(x);
});

foo.subscribe(function(y){
    console.log(y);
});

```

上面2种情况其实是由于函数和Observable都是一种惰性的计算。如果你不调用函数，那么`console.log('Hello')`就不会被执行，也不会有输出。同样的，只用当“调用”（通过`subscribe`订阅）Observable，才会有输出。

此外，调用和订阅都是孤立运行的。2个函数分别被调用会产生2个效果，2个Observable订阅者会产生2个独立的效果。
不像`EventEmitter`会派发它的状态并且不论是否有订阅者都会执行那样，Observable不会派发状态而且是惰性的。

>订阅observable，类似于调用一个函数

有些声音宣称Observable是异步的，这是不对的。如果你使用一个函数来输出日志，像这样:

```js
console.log('before');
console.log(foo.call());
console.log('after');
```
输出是:

```
"before"
"Hello"
42
"after"
```

同样的行为，使用Observable：

```js
console.log('before');
foo.subscribe(function(x){
    console.log('Hello');
    return x;
});
console.log('after');
```
输出是一样的:

```
"before"
"Hello"
42
"after"
```

以上证明了订阅`foo`之后，效果和函数一样，都是同步的。

> 无论是同步方式还是异步方式，obsverable都可以择其一来传递返回值。

Observable和函数之间有什么区别？Observables可以随着时间线返回多个值，函数却不能，因此你可以这样:

```js
function foo() {
  console.log('Hello');
  return 42;
  return 100; // dead code. will never happen
}
```
函数返回单一的值，Observables却可以这样：

```js
var foo = Rx.Observable.create(function(observer){
    console.log('hello');
    observer.next(42);
    observer.next(100);
    observer.next(200);
});

console.log('before');
foo.subscribe(function(x){
    console.log(x);
});
console.log('after');

```

同步式的输出:

```
"before"
"Hello"
42
100
200
"after"
```

也可以使用异步式的输出:

```js
var foo = Rx.Observable.create(function(observer){
    console.log('Hello');
    observer.next(42);
    observer.next(100);
    observer.next(200);
    
    setTimeout(function(){
        observer.next(300);
    },1000);
});

console.log('before');
foo.subscribe(function(x){
    console.log(x);
});
console.log('after');

```

输出为:

```
"before"
"Hello"
42
100
200
"after"
300
```

结论:

- `func.call()`意味着“同步的返回一个值”。
- `observable.subscribe()`意思是“返回任意数量的值，同步式、异步式二择其一。”

## 剖析Observable

`Rx.Observalbe.create()`或者创建操作符，可以 ==创建（created）== Observable流。
Observer则可以 ==订阅（subscribed）== 这个流。
通过 ==执行（execute）== `next()`、`error()`和`complete()`可以向订阅者推送不同的通知。
之后，执行过程可能被 ==处理掉（disposed）== 。
这四个方面都被集成在`Observable`实例当中，但是也有一些方面与其他类型有关，比如`Observer`和`Subscription`。

Observable的核心关注点是:

- 创建Observable流
- 订阅Observable流
- 执行Observable流
- 终止Observable流

###  创建Observable流

`Rx.Observable.create`可以说是Observable构造函数的别名，他可以接受一个参数:`subscribe`函数。

以下的例子创建了一个Observable流，每秒钟向`Observer`发出一个字符串类性值`hi`。

```js
var observable = Rx.Observable.create(function subscribe(observer) {
  var id = setInterval(() => {
    observer.next('hi')
  }, 1000);
});

```

> Observables流可以使用`create()`创建，但是通常我们会使用所谓的创建操作符，像`of()`,`from()`,`interval()`等等。


在上面的例子中，订阅函数(subscribe function)是描述Observalbe最重要的部分。那么，让我来看看何谓订阅。

### 订阅Observable流

在例子中，Observalbe的实例`observable`可以被订阅，像这样:

```js
observable.subscribe(x => console.log(x));
```

也许你会注意到，`observable.subscribe()`和`subscribe`函数在`Rx.Observable.create(function subscribe(observer){...})`中使用了相同的名字，这并不是巧合。
在库中，他们是不同的，但在实际使用中，你可以认为他们在概念上是相等的。

Observable不在多个Observer之间共享`subscribe`。当调用`observable.subscribe()`并得到观察者时，在`Rx.Observable.create(function subscribe(observer){...})`中传入的函数将会被执行。每次执行`observable.subscribe()`都会触发一个单独针对当前Observer的运行逻辑。

>订阅一个Observable流就像调用一个函数，流中的数据将会被传递给回调函数中。

一个`subscribe`函数被调用将会开启一个Observable执行流(Observable execution)，向观察者们输出流中的值或者事件。

### 执行Observable流

代码`Rx.Observable.create(function subscribe(observer){...})`代表了一个“Observable流”，由于惰性计算，只用当有Observer订阅流时，函数才会被执行。
执行过程中随着时间线产生多个数据，方式是同步或异步二选一。

有三个类型的值会在执行流中发出:

- `"Next"` 通知：发出一个值，比如数字，字符串，对象等等。
- `"Error"`通知:发出一个js错误或者异常。
- `Complete`通知：不发出任何值，表示流的结束。

`Next`通知是最重要也是最常用的类型：他代表了实际推送给Observer的值。`Error`和`Complete`通知只会在执行流中发出一次，要么是`Error`，要么是`Complete`。

用正表达式的规则可以很好的表达这种所谓的Observable语法和约定：

```js
next*(error|complete)?
```

>在一个Observable执行流中，会发出0到无限个`Next`通知。而一旦`Error`或者`Complete`通知被发出，执行流将不会再推送任何消息。

下面的例子展示了一个推送了3个`Next`并`Complete`的流：

```js
var observable = Rx.Observable.create(function subscribe(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
});

```

Observables会严格遵守Observable约定，所以下面的代码将不会推送值`4`:

```js
var observable = Rx.Observable.create(function subscribe(observer) {
  observer.next(1);
  observer.next(2);
  observer.next(3);
  observer.complete();
  observer.next(4); // Is not delivered because it would violate the contract
});

```

在订阅函数中使用`try/catch`捕获可能抛出的异常，也是一个很不错的做法：

```js

var observable = Rx.Observable.create(function subscribe(observer) {
  try {
    observer.next(1);
    observer.next(2);
    observer.next(3);
    observer.complete();
  } catch (err) {
    observer.error(err); // delivers an error if it caught one
  }
});
```

### 终止Observable流

Observable流的执行时间线可能是无限长的，但通常我们只用到有限的时间段和观察者处理业务，因此，我们需要一种中断流执行的API。
由于一个执行过程对于每个Observer是独有的，一旦Observer接收到值，那么也必然需要一种中断执行的方式，从而可以节省计算性能和内存空间。

当`observable.subscribe()`被调用，Observer将被附加到新创建的Observable执行过程中，同时返回了一个对象,`Subscription`：

```js

var subscription = observable.subscribe(x => console.log(x));
```

`Subscription`代表了一个持续执行的过程，并且有一套最小化的API允许你中断流的执行过程。可以从这里进一步了解[Subscription类型](http://reactivex.io/rxjs/manual/overview.html#subscription)。以下例子展示了使用`subscription.unsubscribe()`中断持续执行的过程：

```js

var observable = Rx.Observable.from([10, 20, 30]);
var subscription = observable.subscribe(x => console.log(x));
// Later:
subscription.unsubscribe();
```

>当你订阅流就可以获取一个Subscription，代表了持续执行的过程。调用`unsubscribe()`就可以中断执行过程。

当我们使用`create()`创建一个Observable流时，每一个Observable都必须定义它如何处理获取到的资源的处理方式。你可以通过在`subscribe()`函数中返回一个自定义的`unsubscribe`函数，达到这个目的。

举个例子，以下展示了如何中断一个使用`setInterval()`执行`interval`的过程:

```js
var observable = Rx.Observable.create(function subscribe(observer) {
  // Keep track of the interval resource
  var intervalID = setInterval(() => {
    observer.next('hi');
  }, 1000);

  // Provide a way of canceling and disposing the interval resource
  return function unsubscribe() {
    clearInterval(intervalID);
  };
});

```

就像`observable.subscribe()`类似`Observable.create(function subscribe(){..})`一样，我们从`subscribe()`返回的`unsubscribe()`也概念性的等同于`subscription.unsubscribe()`。
事实上，如果我们移除与响应式编程相关的概念，剩下的就是直白的js代码了：

```js

function subscribe(observer) {
  var intervalID = setInterval(() => {
    observer.next('hi');
  }, 1000);

  return function unsubscribe() {
    clearInterval(intervalID);
  };
}

var unsubscribe = subscribe({next: (x) => console.log(x)});

// Later:
unsubscribe(); // dispose the resources

```

我们使用Rx，包括`Observable`、`Observer`和`Subscription`，其原因就是为了使用这些安全(就如Observable约定的)和可组合的操作符。