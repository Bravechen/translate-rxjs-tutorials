**什么是Subscription?** Subscription是一个对象，表示一种可被处置的资源，通常指代一个Observable流的执行过程。

Subscription有一个重要的方法`unsubscribe()`，不需要参数，仅仅用来释放掉subscription实例所持有的的资源。
在之前版本中的RxJS，Subscription被称为“可被处置的”。

```js

var observable = Rx.Observable.interval(1000);
var subscription = observable.subscribe(x => console.log(x));
// Later:
// This cancels the ongoing Observable execution which
// was started by calling subscribe with an Observer.
subscription.unsubscribe();

```

Subscription本质是一个含有`unsubscribe()`方法，用来释放资源或者取消Observable流执行的对象。

多个Subscription可以被组合在一起，从而使调用其中一个Subscription的`unsubscribe()`方法能够让所有的Subscription都取消流的执行。要做到这一点，可以将一个subscription实例“添加”到另一个中去：


```js

var observable1 = Rx.Observable.interval(400);
var observable2 = Rx.Observable.interval(300);

var subscription = observable1.subscribe(x => console.log('first: ' + x));
var childSubscription = observable2.subscribe(x => console.log('second: ' + x));

subscription.add(childSubscription);

setTimeout(() => {
  // Unsubscribes BOTH subscription and childSubscription
  subscription.unsubscribe();
}, 1000);

```

执行一下，我们可以看到输出是这样的:

```
second: 0
first: 0
second: 1
first: 1
second: 2
```

Subscription也有一个名为`remove(otherSubscription)`的方法，用来撤销已经添加到其中的其他Subscription。