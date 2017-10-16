# Operators

RxJS提供的操作符非常有用，尽管Observable是基础对象。
操作符是声明式编程中将复杂的异步代码转变为简单的代码组合的重要措施。

## 什么是操作符 (operator)？

操作符是Observable类型的一组方法，比如像:`.map(...)`,`.filter(...)`,`.merge(...)`,等等。当方法被调用的时候，他们不会更改已经存在的Observable流实例，而是返回一个新的Observable流的实例，这个新的对象中的订阅逻辑则是建立在第一个Observable流之上的。

>一个Operator能够在当前的Observable流基础上创建一个新的Observable流。这是一个纯粹的新处理过程：之前的Observable保持不变。

操作符本质上是能够接收一个Observable流作为输入，并返回一个Observable流作为输出的纯函数。
订阅输出的Observable流也会同时订阅输入的Observable流。
在下面的例子中，我们创建了一个自定义的操作符函数可以将所有输入的值都乘以10，然后输出：

```js

function multiplyByTen(input) {
  var output = Rx.Observable.create(function subscribe(observer) {
    input.subscribe({
      next: (v) => observer.next(10 * v),
      error: (err) => observer.error(err),
      complete: () => observer.complete()
    });
  });
  return output;
}

var input = Rx.Observable.from([1, 2, 3, 4]);
var output = multiplyByTen(input);
output.subscribe(x => console.log(x));
```
输出为:

```
10
20
30
40
```

注意以上的例子中，订阅输出流也会同时订阅输入流的情况。我们称这种现象为：“操作符订阅链”。

## 实例操作符 vs 静态操作符


**什么是实例操作符？** 通常提及操作符，我们假定指的都是实例操作符，他们是Observable流的实例方法。
例如，如果`multiplyByTen`是正式的实例操作符，他将会是这样的：

```js

Rx.Observable.prototype.multiplyByTen = function multiplyByTen() {
  var input = this;
  return Rx.Observable.create(function subscribe(observer) {
    input.subscribe({
      next: (v) => observer.next(10 * v),
      error: (err) => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

```

>实例操作符本质是一个函数，他在内部使用`this`关键字指代Observable输入流。

注意输入Observable流已经不再作为函数的参数，它被假定为this所指向的对象。下面是我们如何使用实例操作符：

```js

var observable = Rx.Observable.from([1, 2, 3, 4]).multiplyByTen();

observable.subscribe(x => console.log(x));

```

**什么是静态操作符？** 有别于实例操作符，静态操作符直接是Observable类的方法。静态操作符函数内部不再使用this，它完全依赖函数的参数。

>静态操作符是依赖于Observable类的一组纯函数，通常被用来从头创建Observable流。

最常见的静态操作符类型是所谓的创建操作符。不同于将输入流转换成输出流的操作符，他们不用接收Observable类型的参数，而是接收诸如`number`类型的参数，就可以创建一道数据流。

一个经典的例子是使用静态操作符`interval()`。他接收一个数字(而不是Observable流)作为输入参数，之后产生一道Observable流作为输出：

```js

var observable = Rx.Observable.interval(1000 /* number of milliseconds */);
```

另一个创建操作符的例子是`create()`，在之前的例子里已经用了很多次。你可以从这里查看所有的[创建操作符](http://reactivex.io/rxjs/manual/overview.html#creation-operators)。

然而，静态操作符们并不仅限于简单创建。一些组合操作符也是静态的，例如`merage`,`combineLates`,`concat`等等。他们可以整合多道输入的Observable流，所以也是非常有意义的静态操作符。

```js

var observable1 = Rx.Observable.interval(1000);
var observable2 = Rx.Observable.interval(400);

var merged = Rx.Observable.merge(observable1, observable2);

```

## 珠宝图 Marble diagrams

为了更好的展示操作符是如何工作的，只有文字性的解释是不够的。很多操作符依赖于时间，例如他们可能会使用延迟，取样，节流，又或者去抖等等方式。
画一些图表能很好的说明这些过程。珠宝图就是用来展示这些的图表，他包含输入Observable流，操作符和相关参数，还有输出流。

>在珠宝图中，时间的流逝被标为从左到右的水平线，线上的每个值(“珠宝”)代表了Observable流在执行过程中发出的值。


下图你可以看到对珠宝的详解。


![image](http://reactivex.io/rxjs/manual/asset/marble-diagram-anatomy.svg)

贯穿本站的文档，我们会广泛的使用珠宝图去解释操作符是如何生效的。在别的场景下他们或许同样有用，例如在whiteboard上或者在单元测试中(作为ASCII图表)。