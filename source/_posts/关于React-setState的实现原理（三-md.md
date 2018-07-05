---
title: 关于React setState的实现原理（三).md
date: 2017-09-10 10:30:51
tags:
---
[上一篇文章中](https://yidongying.github.io/2017/09/08/%E5%85%B3%E4%BA%8EReact-setState%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%EF%BC%88%E4%BA%8C-md/)提到事务即将结束时，会去调用FLUSH_BATCHED_UPDATES的flushBatchedUpdates方法执行批量更新，该方法会去遍历dirtyComponents，对每一项执行performUpdateIfNecessary方法，该方法代码如下：



```
performUpdateIfNecessary: function (transaction) {
    if (this._pendingElement != null) {
      ReactReconciler.receiveComponent(this, this._pendingElement, transaction, this._context);
    } else if (this._pendingStateQueue !== null || this._pendingForceUpdate) {
      this.updateComponent(transaction, this._currentElement, this._currentElement, this._context, this._context);
    } else {
      this._updateBatchNumber = null;
    }
  }
```


 

在我们的setState更新中，其实只会用到第二个  **this._pendingStateQueue !== null**  的判断，即如果**_pendingStateQueue**中还存在未处理的**state**，那就会执行**updateComponent**完成更新。
那**_pendingStateQueue**是何时被处理的呢，继续看！

通过翻阅**updateComponent**方法，我们可以知道**_pendingStateQueue**是在该方法中由**_processPendingState(nextProps, nextContext)**方法做了一些处理，该方法传入两个参数，新的props属性和新的上下文环境，这个上下文环境可以先不用管。我们看看**_processPendingState**的具体实现：


```
_processPendingState: function (props, context) {
    var inst = this._instance;    // _instance保存了Constructor的实例，即通过ReactClass创建的组件的实例
    var queue = this._pendingStateQueue;
    var replace = this._pendingReplaceState;
    this._pendingReplaceState = false;
    this._pendingStateQueue = null;
    if (!queue) {
      return inst.state;
    }
    if (replace && queue.length === 1) {
      return queue[0];
    }
    var nextState = _assign({}, replace ? queue[0] : inst.state);
    for (var i = replace ? 1 : 0; i < queue.length; i++) {
      var partial = queue[i];
      _assign(nextState, typeof partial === 'function' ? partial.call(inst, nextState, props, context) : partial);
    }
    return nextState;
  }，
```


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;什么replace啊什么的都可以暂时不用看，主要先看for循环内部做的事情，replace我们暂时认为是false。
for循环遍历了**_pendingStateQueue**中所有保存的状态，对于每一个状态进行处理，处理时首先判断保存的是function还是object。若是function，就在inst的上下文中执行该匿名函数，该函数返回一个代表新state的object，然后执行assign将其与原有的state合并；若是object，则直接与state合并。
<br>
注意，传入setState的第一个参数如果是function类型，我们可以看到，其第一个参数nextState即表示更新之前的状态；第二个参数props代表更新之后的props，第三个context代表新的上下文环境。之后返回合并后的state。
<br>
这里还需要注意一点，这一点很关键，代码中出现了**this._pendingStateQueue = null**这么一段，这也就意味着**dirtyComponents**进入下一次循环时，执行**performUpdateIfNecessary**不会再去更新组件，这就实现了批量更新，即只做一次更新操作，React在更新组件时就是用这种方式做了优化。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，回来看我们的案例，当我们传入函数作为setState的第一个参数时，我们用该函数提供给我们的state参数来访问组件的state。该state在代码中就对应nextState这个值，这个值在每一次for循环执行时都会对其进行合并，因此第二次执行setState，我们在函数中访问的state就是第一次执行setState后已经合并过的值，所以会打印出2。然而直接通过this.state.count来访问，因为在执行对_pendingStateQueue的for循环时，组件的update还未执行完，this.state还未被赋予新的值，其实了解一下updateComponent会发现，this.state的更新会在_processPendingState执行完执行。所以两次setState取到的都是this.state.count最初的值0，这就解释了之前的现象。其实，这也是React为了解决这种前后state依赖但是state又没及时更新的一种方案，因此在使用时大家要根据实际情况来判断该用哪种方式传参。

接下来我们再来看看setState的第二个参数，回调函数，它是在什么时候执行的。


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
        setTimeout(function() {
            me.setState({count: me.state.count + 1}, function() {
                console.log('did callback');
            });
            console.log('hello');
        }, 0);
    }

    componentDidUpdate() {
        console.log('did update');
    }

    render() {
        return <h1>{this.state.count}</h1>
    }
}
```

这个案例控制台打印顺序是怎样的呢？
不卖关子了，答案是did update，did callback，hello。这里是在一个setTimeout中执行了setState，因此其处于一个单独的事务之中，所以hello最后打印容易理解。然后我们来看看setState执行更新时做了些啥。前面我们知道在执行完组件装载即调用了componentDidMount之后，事务开始执行一系列close方法，这其中包括调用FLUSH_BATCHED_UPDATES中的flushBatchedUpdates，我们来看看这段代码。


```
var flushBatchedUpdates = function () {
  // ReactUpdatesFlushTransaction's wrappers will clear the dirtyComponents
  // array and perform any updates enqueued by mount-ready handlers (i.e.,
  // componentDidUpdate) but we need to check here too in order to catch
  // updates enqueued by setState callbacks and asap calls.
  while (dirtyComponents.length || asapEnqueued) {
    if (dirtyComponents.length) {
      var transaction = ReactUpdatesFlushTransaction.getPooled();
      transaction.perform(runBatchedUpdates, null, transaction);    // 处理批量更新
      ReactUpdatesFlushTransaction.release(transaction);
    }

    if (asapEnqueued) {
      asapEnqueued = false;
      var queue = asapCallbackQueue;
      asapCallbackQueue = CallbackQueue.getPooled();
      queue.notifyAll();    // 处理callback
      CallbackQueue.release(queue);
    }
  }
};
```


 

可以看我做了中文标注的两个地方，这个方法其实主要就是处理了组件的更新和callback的调用。组件的更新发生在runBatchedUpdates这个方法中，下面的queue.notifyAll内部其实就是从队列中去除callback调用，因此应该是先执行完更新，调用componentDidUpdate方法之后，再去执行callback，就有了我们上面的结果。
<br>
**总结**
React在组件更新方面做了很多优化，这其中就包括了上述的批量更新。在componentDidMount中执行了N个setState，如果执行N次更新是件很傻的事情。React利用其独特的事务实现，做了这些优化。正是因为这些优化，才造成了上面见到的怪现象。还有一点，再使用this.state时一定要注意组件的生命周期，很多时候在获取state的时候，组件更新还未完成，this.state还未改变，这是很容易造成bug的一个地方，要避免这个问题，需要对组件生命周期有一定的了解。