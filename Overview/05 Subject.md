**Subject是什么？** RxJS的Subject是Observable的一个特殊类型，他可以将流中的值广播给众多观察者(Observer)。
一般的Observalbe流是单一广播制(每一个订阅流的Observer拥有一个独立的执行过程)。

>一个Subject类似一道Observable数据流，但是可以对多个Observer进行多点广播。这就像事件触发器(EventEmitter)：维护了一个侦听器的列表。

**每一个Subject就是一个Observable流。** 对于给定的Subject，你可以订阅它(`subscribe`)，提供一个Observer，之后将会正常的接收传递来的数据。从Observer的角度来说，它是无法分辨一个流中的值是来源于单一广播机制的Observable流还是一个Subject流。

在Subject内部，订阅(`subscribe`)不会引起一个新的接收数据的过程。类似于其他库或语言中的注册事件侦听器(`addListener`)，它会直接把给定的Observer放入到一个注册列表中。

**每一个Subject也是一个观察者(`Observer`)。** 拥有`next(v)`、`error(e)`和`complete()`方法。往Subject中填充数据，只需要调用`next(theValue)`即可，它将会把数据广播给所有已注册的Observer。 

以下的例子中，我们设定了2个订阅Subject流的Observer，然后我们填充一些数据到Subject：

```js

var subject = new Rx.Subject();

subject.subscribe({
  next: (v) => console.log('observerA: ' + v)
});
subject.subscribe({
  next: (v) => console.log('observerB: ' + v)
});

subject.next(1);
subject.next(2);

```

得到了如下输出:

```
observerA: 1
observerB: 1
observerA: 2
observerB: 2
```

因为Subject是一个Observer，因此你也可以将它作为任何Observable的`subscribe()`的参数，订阅这个Observable流，就像下面这样：

```js

var subject = new Rx.Subject();

subject.subscribe({
  next: (v) => console.log('observerA: ' + v)
});
subject.subscribe({
  next: (v) => console.log('observerB: ' + v)
});

var observable = Rx.Observable.from([1, 2, 3]);

observable.subscribe(subject); // You can subscribe providing a Subject

```

运行的结果：

```
observerA: 1
observerB: 1
observerA: 2
observerB: 2
observerA: 3
observerB: 3
```

在上面的方法中，我们使用Subject将一个单点广播的Observable流转换为多点广播。这也佐证了，Subject是可以将任何Observable流共享给多个Observer的唯一途径。

除了Subject，还有一些衍生出的专门的Subject：`BehaviorSubject`,`ReplaySubject`和`AsyncSubject`。

## 多路传播的Observable流 Multicasted Observables

相比于只能推送消息给单个的Observer的“单路Observable流”，利用具有多个订阅者的Subject，“多路传播的Observable流”可以有多个通知通道。 

>多路传播的Observable在后台通过使用Subject让多个Observers能够从同一个Observable流中获取数据。

在后台，multicast操作符是这样工作的：Obersver订阅潜在的Subject,而Subject又订阅了源Observable流。下面的例子和之前使用`observable.subscribe(subject)`的情况类似：

```js

var source = Rx.Observable.from([1, 2, 3]);
var subject = new Rx.Subject();
var multicasted = source.multicast(subject);

// These are, under the hood, `subject.subscribe({...})`:
multicasted.subscribe({
  next: (v) => console.log('observerA: ' + v)
});
multicasted.subscribe({
  next: (v) => console.log('observerB: ' + v)
});

// This is, under the hood, `source.subscribe(subject)`:
multicasted.connect();

```

multicast流返回了一个看似普通的Observable流，但是当订阅的时候他表现的与Subject类似。这个流被称作`ConnectableObservable`流，本质是一个Observable流，但拥有`connect()`方法。

`connect()`在内部执行了`source.subscribe(subject)`,并且返回了一个你可以取消Observable流执行的`Subscription`。因此，当可被共享的Observable流开始时，`connect()`方法对于精确的判定执行过程很重要。

### 引用计数 Reference counting

手动的调用`connect()`和执行`Subscription`往往是很累人的。我们当然希望可以在第一个Observer订阅的时候就自动的执行`connect()`，并且最好在最后一个Observer取消订阅(unsubscribe)的时候能自动取消流的执行。

考虑一下，处于下列操作顺序时的表现情况：

1. 第一个Observer订阅了多路传播的Observable流
2. 多路传播的Observable流呈被连接状态
3. 调用`next()`传0给第一个Observer
4. 第二个Observer订阅多路传播Observable流
5. 调用`next()`传1给第一个Observer
6. 调用`next()`传1给第二个Observer
7. 第一个Observer取消订阅
8. 调用`next()`传2给第二个Observer
9. 第二个Observer取消订阅
10. 多路传播Observable流的连接情况是未被订阅状态

为了显式的调用`connect()`实现这个过程，我们编写如下代码:

```js
var source = Rx.Observable.interval(500);
var subject = new Rx.Subject();
var multicasted = source.multicast(subject);
var subscription1, subscription2, subscriptionConnect;

subscription1 = multicasted.subscribe({
  next: (v) => console.log('observerA: ' + v)
});
// We should call `connect()` here, because the first
// subscriber to `multicasted` is interested in consuming values
subscriptionConnect = multicasted.connect();

setTimeout(() => {
  subscription2 = multicasted.subscribe({
    next: (v) => console.log('observerB: ' + v)
  });
}, 600);

setTimeout(() => {
  subscription1.unsubscribe();
}, 1200);

// We should unsubscribe the shared Observable execution here,
// because `multicasted` would have no more subscribers after this
setTimeout(() => {
  subscription2.unsubscribe();
  subscriptionConnect.unsubscribe(); // for the shared Observable execution
}, 2000);
```

如果我们想避免显式的调用`connect()`，我们可以使用ConnectableObservable的`refCount()`方法(引用计数)，他返回了一个存有众多订阅者的Observable流。当订阅者的数量从0增加到1时，将会自动调用`connect()`，开始共享流。
当订阅者的数量从1变为0，即将处于未订阅状态时，将会自动停止下一步的执行。

>`refCount`使多路传播Observable流在第一个订阅者出现时自动启动，在最后一个订阅者离开时自动停止。

请看下面的例子:

```js

var source = Rx.Observable.interval(500);
var subject = new Rx.Subject();
var refCounted = source.multicast(subject).refCount();
var subscription1, subscription2, subscriptionConnect;

// This calls `connect()`, because
// it is the first subscriber to `refCounted`
console.log('observerA subscribed');
subscription1 = refCounted.subscribe({
  next: (v) => console.log('observerA: ' + v)
});

setTimeout(() => {
  console.log('observerB subscribed');
  subscription2 = refCounted.subscribe({
    next: (v) => console.log('observerB: ' + v)
  });
}, 600);

setTimeout(() => {
  console.log('observerA unsubscribed');
  subscription1.unsubscribe();
}, 1200);

// This is when the shared Observable execution will stop, because
// `refCounted` would have no more subscribers after this
setTimeout(() => {
  console.log('observerB unsubscribed');
  subscription2.unsubscribe();
}, 2000);

```

执行过后的输出是:

```
observerA subscribed
observerA: 0
observerB subscribed
observerA: 1
observerB: 1
observerA unsubscribed
observerB: 2
observerB unsubscribed
```

`refCount()`方法只存在于`ConnectableObservable`中，他返回一个Observable流，而不是另一个ConnectableObservable流。

## BehaviorSubject

`BehaviorSubject`是一类特异的Subject。具有返回“当前值”的特性。它存储了流中最新的值并把它推送给自己的用户，不论它的新旧与否，都能够立即收到推送的这个“当前值”。

>BehaviorSubject 非常有利于表示“变化中的值”。举例来说，每年都有生日是一道Subject数据流，但是一个人的年龄却是一个BehaviorSubject流。

来看下面的例子，BehaviorSubject以0为值进行初始化，第一个订阅的Observer将会直接收到这个值。当2被填充入流之后，第二个Observer订阅流时，尽管时间较晚，也会收到最新值2。

```js

var subject = new Rx.BehaviorSubject(0); // 0 is the initial value

subject.subscribe({
  next: (v) => console.log('observerA: ' + v)
});

subject.next(1);
subject.next(2);

subject.subscribe({
  next: (v) => console.log('observerB: ' + v)
});

subject.next(3);

```

输出如下：

```
observerA: 0
observerA: 1
observerA: 2
observerB: 2
observerA: 3
observerB: 3
```

## ReplaySubject

ReplaySubject 很像BehaviorSubject，他会把时间线中较老的值推送给新的订阅者们，而且他还可以记录Observable流中一段时间的值。

>ReplaySubject能够记录Observable流中的多个值，并将它们推送给新的订阅者。

创建ReplaySubject时，你可以指定需要回放多少个值，像这样：

```js

var subject = new Rx.ReplaySubject(3); // buffer 3 values for new subscribers

subject.subscribe({
  next: (v) => console.log('observerA: ' + v)
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log('observerB: ' + v)
});

subject.next(5);

```

输出如下:

```

observerA: 1
observerA: 2
observerA: 3
observerA: 4
observerB: 2
observerB: 3
observerB: 4
observerA: 5
observerB: 5
```
在设定数据量大小之外，你还可以指定一个以毫秒为单位的窗口时间，用来确定记录的数据所在的时间区间（数据有多老）。
在下面的例子中，我们使用了一个较大的数据量设定，同时还设定了500毫秒的窗口时间。

```js

var subject = new Rx.ReplaySubject(100, 500 /* windowTime */);

subject.subscribe({
  next: (v) => console.log('observerA: ' + v)
});

var i = 1;
setInterval(() => subject.next(i++), 200);

setTimeout(() => {
  subject.subscribe({
    next: (v) => console.log('observerB: ' + v)
  });
}, 1000);

```

运行结果显示，第二个Observer在订阅之后，获得了数据流中最后500毫秒事件内产生的3,4和5三个值。

```js
observerA: 1
observerA: 2
observerA: 3
observerA: 4
observerA: 5
/************/
observerB: 3
observerB: 4
observerB: 5
/************/
observerA: 6
observerB: 6
...

```

## AsyncSubject

AsyncSubject是Subject的另一个变化，他会在流发出`complete`通知时，将数据流中的最后一个值推送给所有订阅流的Observer。

```js

var subject = new Rx.AsyncSubject();

subject.subscribe({
  next: (v) => console.log('observerA: ' + v)
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log('observerB: ' + v)
});

subject.next(5);
subject.complete();

```
输出为:
With output:
```
observerA: 5
observerB: 5
```

AsyncSubject非常类似`last()`操作符，它会等待`complete`通知，并在那时推送流中的数据值。