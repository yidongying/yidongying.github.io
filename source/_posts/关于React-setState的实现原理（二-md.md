---
title: 关于React setState的实现原理（二).md
date: 2017-09-08 09:57:48
tags:
categories: react的概念理解
---
上一篇文章中,写到了关于Batch Update的实现,有不懂的童鞋可以回头看看 [上一篇文章](https://yidongying.github.io/2017/09/05/%E5%85%B3%E4%BA%8EReact-setState%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%EF%BC%88%E4%B8%80-md/)
**React中的Transaction**
大家学过sql server的都知道我们可以批量处理sql语句，原理其实都是基于上一篇我们说的Datch Update机制。当所有的操作均执行成功，才会执行修改操作；若有一个操作失败，则执行rollback（回滚）。

在React中，我们介绍过事件会在函数前后执行自己的逻辑，具体就是调用perform方法进入一个事件，这个方法会传入一个method参数。执行perform时先执行initializeAll方法按照一定顺序执行一系列的initialize操作，然后执行传入的method，method执行完后，就执行closeAll方法按照一定顺序执行一系列的close操作。注意一种事件不能同时开启，否则会抛出异常。给一个例子是实现事件：



```
var Transaction = require('./Transaction');

// 我们自己定义的 Transaction
var MyTransaction = function() {
  // do sth.
};

Object.assign(MyTransaction.prototype, Transaction.Mixin, {
  getTransactionWrappers: function() {
    return [{
      initialize: function() {
        console.log('before method perform');
      },
      close: function() {
        console.log('after method perform');
      }
    }];
  };
});

var transaction = new MyTransaction();
var testMethod = function() {
  console.log('test');
}
transaction.perform(testMethod);

// before method perform
// test
// after method perform
```


 

具体到实现上，React 中的 Transaction 提供了一个 Mixin 方便其它模块实现自己需要的事务。而要使用 Transaction 的模块，除了需要把 Transaction 的 Mixin 混入自己的事务实现中外，还需要额外实现一个抽象的 getTransactionWrappers 接口。这个接口是 Transaction 用来获取所有需要封装的前置方法（initialize）和收尾方法（close）的，因此它需要返回一个数组的对象，每个对象分别有 key 为 initialize 和 close 的方法。

当然在实际代码中 React 还做了异常处理等工作，这里不详细展开。有兴趣的同学可以参考源码中 Transaction 实现。

 

组件调用ReactDOM.render()之后，会执行一个_renderNewRootComponent的方法，大概是该方法执行了一个ReactUpdates.batchedUpdates()。 那么batchedUpdates是什么呢？


```
var transaction = new ReactDefaultBatchingStrategyTransaction();

var ReactDefaultBatchingStrategy = {
  isBatchingUpdates: false,

  /**
   * Call the provided function in a context within which calls to `setState`
   * and friends are batched such that components aren't updated unnecessarily.
   */
  batchedUpdates: function (callback, a, b, c, d, e) {
    var alreadyBatchingUpdates = ReactDefaultBatchingStrategy.isBatchingUpdates;

    ReactDefaultBatchingStrategy.isBatchingUpdates = true;

    // The code is written this way to avoid extra allocations
    if (alreadyBatchingUpdates) {
      return callback(a, b, c, d, e);
    } else {
      return transaction.perform(callback, null, a, b, c, d, e);
    }
  }
};
```

 

从代码中可以看出，这个batchedUpdate是第一次调用alreadyBatchingUpdates是false（储存起来了），回去执行transaction.perform(method)（前边说过perform执行会进入一个时间），这样就进入第一个事务，

这个事务是啥我们现在不用管，我们只需要知道这个transaction是ReactDefaultBatchingStrategyTransaction的实例，它代表了其中一类事务的执行。然后会执行method方法，就会进行组件的首次装载。完成后会调用

componentDidMount(注意，此时还是在执行method方法，事务还没结束，事务只有在执行完method后执行一系列close才会结束),在该方法中，我们调用了setState，出现了一系列奇怪的现象。因此，我们再来看看

setState方法：


```
ReactComponent.prototype.setState = function (partialState, callback) {
  !(typeof partialState === 'object' || typeof partialState === 'function' || partialState == null) ? "development" !== 'production' ? invariant(false, 'setState(...): takes an object of state variables to update or a function which returns an object of state variables.') : _prodInvariant('85') : void 0;
  this.updater.enqueueSetState(this, partialState);
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'setState');
  }
};
```


setState在调用时做了两件事，第一，调用enqueueSetState。该方法将我们传入的partialState添加到一个叫做_pendingStateQueue的队列中去存起来，然后执行一个enqueueUpdate方法。
<br>
第二，如果存在callback就调用enqueueCallback将其存入一个_pendingCallbacks队列中存起来。然后我们来看enqueueUpdate方法。



```
function enqueueUpdate(component) {
  ensureInjected();

  // Various parts of our code (such as ReactCompositeComponent's
  // _renderValidatedComponent) assume that calls to render aren't nested;
  // verify that that's the case. (This is called by each top-level update
  // function, like setState, forceUpdate, etc.; creation and
  // destruction of top-level components is guarded in ReactMount.)

  if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }

  dirtyComponents.push(component);
  if (component._updateBatchNumber == null) {
    component._updateBatchNumber = updateBatchNumber + 1;
  }
}
```


 
<br>
上面的代码中，这个**batchingStrategy**就是上面的**ReactDefaultBatchingStrategy**，只是它通过**inject**的形式对其进行赋值，比较隐蔽。因此，我们当前的setState已经处于了这一类事务之中，isBatchingUpdates已经被置为true，所以将会把它添加到**dirtyComponents**中，在某一时刻做批量更新。
<br>
因此在前两个setState中，并没有做任何状态更新，以及组件更新的事，而仅仅是将新的state和该组件存在了队列之中，因此两次都会打印出0，我们之前的第一个问题就解决了，还有一个问题，我们接着往下走。
<br>
在setTimeout中执行的setState打印出了2和3，有了前面的铺垫，我们大概就能得出结论，这应该就是因为这两次setState分别执行了一次完整的事务，导致state被直接更新而造成的结果。那么问题来了，为什么setTimeout中的setState会分别执行两次不同的事务？之前执行ReactDOM.render开启的事务在什么时候结束了？我们来看下列代码。



```
var RESET_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: function () {
    ReactDefaultBatchingStrategy.isBatchingUpdates = false;
  }
};

var FLUSH_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: ReactUpdates.flushBatchedUpdates.bind(ReactUpdates)
};

var TRANSACTION_WRAPPERS = [FLUSH_BATCHED_UPDATES, RESET_BATCHED_UPDATES];

function ReactDefaultBatchingStrategyTransaction() {
  this.reinitializeTransaction();
}

_assign(ReactDefaultBatchingStrategyTransaction.prototype, Transaction, {
  getTransactionWrappers: function () {
    return TRANSACTION_WRAPPERS;
  }
});
```


这段代码也是写在**ReactDefaultBatchingStrategy**这个对象中的。我们之前提到这个事务中**transaction**是**ReactDefaultBatchingStrategyTransaction**的实例，这段代码其实就是给该事务添加了两个在事务结束时会被调用的close方法。即在perform中的method执行完毕后，会按照这里数组的顺序**[FLUSH_BATCHED_UPDATES, RESET_BATCHED_UPDATES]**依次调用其close方法。
<br>
**FLUSH_BATCHED_UPDATES**是执行批量更新操作。**RESET_BATCHED_UPDATES**我们可以看到将**isBatchingUpdates**变回false，即意味着事务结束。


```
function enqueueUpdate(component) {
  ensureInjected();
  // Various parts of our code (such as ReactCompositeComponent's
  // _renderValidatedComponent) assume that calls to render aren't nested;
  // verify that that's the case. (This is called by each top-level update
  // function, like setState, forceUpdate, etc.; creation and
  // destruction of top-level components is guarded in ReactMount.)
  if (!batchingStrategy.isBatchingUpdates) { //上一个事件结束执行过isBatchedUpdates=false,所以进入if中
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }
  dirtyComponents.push(component);
  if (component._updateBatchNumber == null) {
    component._updateBatchNumber = updateBatchNumber + 1;
  }
}
```

 

<br>
接下来再调用setState时（在setTimeout中，前文说过一步操作不会在主线程，我理解是在主线程结束才会执行，此时的主线程事件已经结束），enqueueUpdate不会再将其添加到dirtyComponents中，而是执行batchingStrategy.batchedUpdates(enqueueUpdate, component)开启一个新事务。
<br>
但是需要注意，这里传入的参数是enqueueUpdate，即perform中执行的method为enqueueUpdate，而再次调用该enqueueUpdate方法会去执行dirtyComponents那一步。这就可以理解为，处于单独事务的setState也是通过将组件添加到dirtyComponents来完成更新的，只不过这里是在enqueueUpdate执行完毕后立即执行相应的close方法完成更新，而前面两个setState需在整个组件装载完成之后，即在componentDidMount执行完毕后才会去调用close完成更新。总结一下4个setState执行的过程就是：先执行两次console.log，然后执行批量更新，再执行setState直接更新，执行console.log，最后再执行setState直接更新，再执行console.log，所以就会得出0,0,2,3。
<br>
到现在上面的问题已经解决，但是又出现一个新问题：


```
class Root extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  componentDidMount() {
    let me = this;
    me.setState({
      count: me.state.count + 1
    });
    me.setState({
      count: me.state.count + 1
    });
  }
  render() {
    return (
      <h1>{this.state.count}</h1>   //页面中将打印出1
    )
  }
}
```


```
class Root extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
  componentDidMount() {
    let me = this;
    me.setState(function(state, props) {
      return {
        count: state.count + 1
      }
    });
    me.setState(function(state, props) {
      return {
        count: state.count + 1
      }
    });
  }
  render() {
    return (
      <h1>{this.state.count}</h1>   //页面中将打印出2
    )
  }
}
```


 
<br>
这两种写法，一个是在setState中传入了object，一个是传入了function，却得到了两种不同的结果，这是什么原因造成的，这就需要我们去深入了解一下进行批量更行时都做了些什么。
关于这个部分, 可以看看我的[下一篇文章](https://yidongying.github.io/2018/07/05/%E5%85%B3%E4%BA%8EReact-setState%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%EF%BC%88%E4%B8%89-md/)

