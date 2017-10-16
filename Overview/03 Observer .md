# Observer 观察者

**什么是Observer？** 观察者(Observer)是Observable流推送数据的用户。观察者们(Observers)就是一组callback函数的集合，监听着每一个Observable流推送出的不同类型的通知，包括:`next`,`error`和`complete`。

以下是一个经典的观察者对象:

```js

var observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};

```

为了使用观察者，需要让他订阅一个Observable流：

```js

observable.subscribe(observer);

```

观察者是一个包含三个回调函数的对象，每一个函数都时刻准备接收来自Observable流推送的不同消息。

Observer在RxJS中是被优待的。如果没有为某个类型的通知提供callback，Observable流的执行过程仍然会照常进行，但是响应的通知将会被忽略，因为观察者没有提供相应的callback来接收。

下面是一个Observer没有提供`complete`响应(callback)的例子：

```js
var observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
};
```

订阅一个Observable流的时候，你也可以只提供一个callback函数作为参数，而不用完整提供一个包含三个回调的对象，就像下面的例子:

```js
observable.subscribe(x => console.log('Observer got a next value: ' + x));

```

在`observable.subscribe()`内部，将会创建一个观察者对象(Observer object)，并将第一个参数提供的callback作为`next`通知的响应函数。接受三个类型通知的callback也可以分别以参数的形式提供:

```js

observable.subscribe(
  x => console.log('Observer got a next value: ' + x),
  err => console.error('Observer got an error: ' + err),
  () => console.log('Observer got a complete notification')
);

```