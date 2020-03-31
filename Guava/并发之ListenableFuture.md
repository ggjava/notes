ListenableFuture

并发编程是一个难题，但是一个强大而简单的抽象可以显著的简化并发的编写。出于这样的考虑，Guava定义了`ListenableFuture`接口并继承了JDK concurrent包下的`Future`接口。

我们强烈地建议你在代码中多使用`ListenableFuture`来代替JDK的`Future`

* 大多数`Futures`方法中需要它。
* 转到`ListenableFuture`编程比较容易。
* Guava提供的通用公共类封装了公共的操作方方法，不需要提供Future和`ListenableFuture`的扩展方法。

## 接口

传统JDK中的`Future`通过异步的方式计算返回结果:在多线程运算中可能在没有结束返回结果，`Future`是运行中的多线程的一个引用句柄，确保在服务执行返回一个`Result`。

`ListenableFuture`可以允许你注册回调方法(callbacks)，在运算（多线程执行）完成的时候进行调用, 或者在运算（多线程执行）完成后立即执行。这样简单的改进，使得可以明显的支持更多的操作，这样的功能在JDK concurrent中的`Future`是不支持的。

`ListenableFuture`中的基础方法是[addListener(Runnable, Executor)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFuture.html#addListener-java.lang.Runnable-java.util.concurrent.Executor-), 该方法会在多线程运算完的时候，指定的`Runnable`参数传入的对象会被指定的`Executor`执行。

## 添加回调（Callbacks）

多数用户喜欢使用[Futures.addCallback(ListenableFuture<V>, FutureCallback<V>, Executor)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#addCallback-com.google.common.util.concurrent.ListenableFuture-com.google.common.util.concurrent.FutureCallback-java.util.concurrent.Executor-)的方式, 或者另外一个版本，默认是采用MoreExecutors.sameThreadExecutor()线程池, 为了简化使用，Callback采用轻量级的设计. [FutureCallback<V>](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html)中实现了两个方法:

* [onSuccess(V)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html#onSuccess-V-),在`Future`成功的时候执行，根据`Future`结果来判断。
* [onFailure(Throwable)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html#onFailure-java.lang.Throwable-), 在 `Future`失败的时候执行，根据`Future`结果来判断。

## ListenableFuture的创建

对应JDK中的[ExecutorService.submit(Callable)](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html#submit-java.util.concurrent.Callable-)提交多线程异步运算的方式，Guava提供了[ListeningExecutorService](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListeningExecutorService.html)接口, 该接口返回`ListenableFutur`e而相应的`ExecutorService`返回普通的`Future`。将`ExecutorService`转为`ListeningExecutorService`，可以使用[MoreExecutors.listeningDecorator(ExecutorService)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/MoreExecutors.html#listeningDecorator-java.util.concurrent.ExecutorService-)进行装饰。

```java
ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));
ListenableFuture<Explosion> explosion = service.submit(new Callable<Explosion>() {
  public Explosion call() {
    return pushBigRedButton();
  }
});
Futures.addCallback(explosion, new FutureCallback<Explosion>() {
  // we want this handler to run immediately after we push the big red button!
  public void onSuccess(Explosion explosion) {
    walkAwayFrom(explosion);
  }
  public void onFailure(Throwable thrown) {
    battleArchNemesis(); // escaped the explosion!
  }
});
```

另外, 假如你是从`FutureTask`转换而来的, Guava提供[ListenableFutureTask.create(Callable)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFutureTask.html#create-java.util.concurrent.Callable-)和[ListenableFutureTask.create(Runnable, V)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFutureTask.html#create-java.lang.Runnable-V-). 和JDK不同的是, `ListenableFutureTask`不能随意被继承（`ListenableFutureTask`中的done方法实现了调用`listener`的操作）。

假如你喜欢抽象的方式来设置future的值，而不是想实现接口中的方法，可以考虑继承抽象类[AbstractFuture<V>](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractFuture.html)或者直接使用[SettableFuture](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/SettableFuture.html)。

假如你必须将其他API提供的Future转换成`ListenableFuture`，你没有别的方法只能采用硬编码的方式 [JdkFutureAdapters.listenInPoolThread(Future)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/JdkFutureAdapters.html)来将Future转换成`ListenableFuture`。尽可能地采用修改原生的代码返回ListenableFuture会更好一些。

## Application

使用`ListenableFuture`最重要的理由是它可以进行一系列的复杂链式的异步操作。

```java
ListenableFuture<RowKey> rowKeyFuture = indexService.lookUp(query);
AsyncFunction<RowKey, QueryResult> queryFunction =
  new AsyncFunction<RowKey, QueryResult>() {
    public ListenableFuture<QueryResult> apply(RowKey rowKey) {
      return dataService.read(rowKey);
    }
  };
ListenableFuture<QueryResult> queryFuture =
    Futures.transformAsync(rowKeyFuture, queryFunction, queryExecutor);
```

其他更多的操作可以更加有效的支持而JDK中的Future是没法支持的.

不同的操作可以在不同的`Executors`中执行，单独的`ListenableFuture`可以有多个操作等待。

当一个操作开始的时候其他的一些操作也会尽快开始执行–“fan-out”–ListenableFuture 能够满足这样的场景：触发所有的回调（callbacks）。反之更简单的工作是，同样可以满足“fan-in”场景，触发`ListenableFuture`获取（get）计算结果，同时其它的`Futures`也会尽快执行：可以参考the implementation of Futures.allAsList 。

| 方法 | 描述 | 参考 |
| - | - | - |
| [transform(ListenableFuture<A\>, AsyncFunction<A, B\>, Executor)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transformAsync-com.google.common.util.concurrent.ListenableFuture-com.google.common.util.concurrent.AsyncFunction-java.util.concurrent.Executor-)* | 返回一个新的 ListenableFuture ，该`ListenableFuture`返回的 result 是由传入的 AsyncFunction 参数指派到传入的 ListenableFuture 中. | [transform(ListenableFuture<A\>, AsyncFunction<A, B\>)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transformAsync-com.google.common.util.concurrent.ListenableFuture-com.google.common.util.concurrent.AsyncFunction-) |
| [transform(ListenableFuture<A\>, Function<A, B\>, Executor)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transform-com.google.common.util.concurrent.ListenableFuture-com.google.common.base.Function-java.util.concurrent.Executor-) | 返回一个新的 ListenableFuture ，该 ListenableFuture 返回的 result 是由传入的Function 参数指派到传入的 ListenableFuture 中. | [transform(ListenableFuture<A\>, Function<A, B\>)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transform-com.google.common.util.concurrent.ListenableFuture-com.google.common.base.Function-) |
| [allAsList(Iterable<ListenableFuture<V\>\>)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#allAsList-java.lang.Iterable-) | 返回一个`ListenableFuture`，该ListenableFuture 返回的 result 是一个 List，List 中的值是每个`ListenableFuture`的返回值，假如传入的其中之一fails或者cancel，这 个Future fails 或者 canceled | [allAsList(ListenableFuture<V\>...)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#allAsList-com.google.common.util.concurrent.ListenableFuture...-) |
| [successfulAsList(Iterable<ListenableFuture<V\>\>)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#successfulAsList-java.lang.Iterable-) | 返回一个 ListenableFuture ，该 Future 的结果包含所有成功的 Future，按照原来的顺序，当其中之一 Failed 或者 cancel，则用 null 替代 | [successfulAsList(ListenableFuture<V\>...)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#successfulAsList-com.google.common.util.concurrent.ListenableFuture...-) |

[AsyncFunction<A, B\>](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AsyncFunction.html)中提供一个方法`ListenableFuture<B> apply(A input)`，它可以被用于异步变换值。

```java
List<ListenableFuture<QueryResult>> queries;
// The queries go to all different data centers, but we want to wait until they're all done or failed.

ListenableFuture<List<QueryResult>> successfulQueries = Futures.successfulAsList(queries);

Futures.addCallback(successfulQueries, callbackOnSuccessfulQueries);
```

## 避免嵌套`Future`

在代码调用泛型接口并返回`Future`的情况下，可能最终使用嵌套的`Future`。例如:

```java
executorService.submit(new Callable<ListenableFuture<Foo>() {
  @Override
  public ListenableFuture<Foo> call() {
    return otherExecutorService.submit(otherCallable);
  }
});
```

将返回一个`ListenableFuture<ListenableFuture<Foo>>`。这段代码是不正确的，因为如果外部future上的cancel与外部future的完成竞争，那么该取消将不会传播到内部future。使用get()或侦听器检查另一个将来的失败也是一个常见的错误，但是除非特别小心，否则会抑制从otherCallable抛出的异常。为了避免这种情况，Guava所有的未来处理方法(有些来自JDK)都有*Async版本，可以安全地打开这个嵌套-

[transform(ListenableFuture<A\>, Function<A, B\>, Executor)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transform-com.google.common.util.concurrent.ListenableFuture-com.google.common.base.Function-java.util.concurrent.Executor-)

[transformAsync(ListenableFuture<A\>,AsyncFunction<A, B\>,Executor)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transformAsync-com.google.common.util.concurrent.ListenableFuture-com.google.common.util.concurrent.AsyncFunction-java.util.concurrent.Executor-)

[ExecutorService.submit(Callable)](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html#submit-java.util.concurrent.Callable-)

[submitAsync(AsyncCallable<A\>, Executor)](https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#submitAsync-com.google.common.util.concurrent.AsyncCallable-java.util.concurrent.Executor-)
