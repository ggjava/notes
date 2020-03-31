## 概述

`java.util.concurrent`包提供了创建并发应用程序的工具。

`java.util.concurrent`并发包含太多的特性，不可能在一篇文章中讨论。在本文中，我们将主要关注这个包中一些最有用的实用工具，如:

* Executor
* ExecutorService
* ScheduledExecutorService
* Future
* CountDownLatch
* CyclicBarrier
* Semaphore
* ThreadFactory
* BlockingQueue
* DelayQueue
* Locks
* Phaser

## Executor

`Executor`是表示执行所提供任务的对象的接口。

任务应该在新线程或当前线程上运行，它取决于特定的实现(从调用开始的地方)。因此，使用这个接口，我们可以将任务执行流与实际的任务执行机制解耦。

这里需要注意的一点是`Executor`并不严格要求任务执行是异步的。在最简单的情况下，执行程序可以在调用线程中立即调用提交的任务。

我们需要创建一个调用程序来创建`executor`实例:

```java
public class Invoker implements Executor {
    @Override
    public void execute(Runnable r) {
        r.run();
    }
}
```

现在，我们可以使用这个调用程序来执行任务。

```java
public void execute() {
    Executor executor = new Invoker();
    executor.execute( () -> {
        // task to be performed
    });
}
```

需要注意的是，如果执行器不能接受任务执行，它将抛出`RejectedExecutionException`。

## ExecutorService

`ExecutorService`是异步处理的完整解决方案。它管理内存中的队列，并根据线程可用性调度提交的任务。

要使用`ExecutorService`，我们需要创建一个`Runnable`类。

```java
public class Task implements Runnable {
    @Override
    public void run() {
        // task details
    }
}
```

创建`ExecutorService`实例并分配此任务。在创建时，需要指定线程池的大小。

```java
ExecutorService executor = Executors.newFixedThreadPool(10);
```

如果想创建一个单线程`ExecutorService`实例，可以使用`newSingleThreadExecutor(ThreadFactory ThreadFactory)`来创建实例。

创建了`executor`，就可以使用它来提交任务。

```java
public void execute() { 
    executor.submit(new Task()); 
}
```

还可以在提交任务时创建`Runnable`实例。

```java
executor.submit(() -> {
    new Task();
});
```

它还提供了两种开箱即用的执行终止方法。

第一个是`shutdown()`;它将等待所有提交的任务完成执行。

另一个方法是`shutdownNow()`，它立即终止所有挂起/执行的任务。

还有另一种方法`awaitTermination(long timeout, TimeUnit)`，它会强制阻塞，直到在触发关机事件或发生执行超时或执行线程本身中断后，所有任务都已完成执行

```java
try {
    executor.awaitTermination( 20l, TimeUnit.NANOSECONDS );
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

## ScheduledExecutorService

`ScheduledExecutorService`是与`ExecutorService`类似的接口，但它可以定期执行任务。

`Executor`和`ExecutorService`的方法在现场调度，不引入任何人为的延迟。`0`或任何`负值`表示需要立即执行请求。

可以使用`Runnable`和`Callable`接口来定义任务。

```java
public void execute() {
    ScheduledExecutorService executorService
      = Executors.newSingleThreadScheduledExecutor();

    Future<String> future = executorService.schedule(() -> {
        // ...
        return "Hello world";
    }, 1, TimeUnit.SECONDS);

    ScheduledFuture<?> scheduledFuture = executorService.schedule(() -> {
        // ...
    }, 1, TimeUnit.SECONDS);

    executorService.shutdown();
}
```

`ScheduledExecutorService`也可以在给定的固定延迟后调度任务:

```java
executorService.scheduleAtFixedRate(() -> {
    // ...
}, 1, 10, TimeUnit.SECONDS);

executorService.scheduleWithFixedDelay(() -> {
    // ...
}, 1, 10, TimeUnit.SECONDS);
```

这里，`scheduleAtFixedRate( Runnable command, long initialDelay, long period, TimeUnit unit )`方法创建并执行一个周期性动作，该动作在提供的初始延迟之后首先调用，然后在给定的时间段内调用，直到服务实例关闭。

`scheduleWithFixedDelay( Runnable command, long initialDelay, long delay, TimeUnit unit )`方法创建并执行一个周期动作，该动作在提供的初始延迟之后首先调用，并在执行的终止和下一个调用之间的给定延迟中重复调用。

## Future

`Future`用于表示异步操作的结果。它提供了一些方法，用于检查异步操作是否完成、获取计算结果等。

而且，`cancel(boolean mayInterruptIfRunning)` API取消操作并释放执行线程。如果`mayInterruptIfRunning`的值为真，则执行该任务的线程将立即终止。

否则，将允许正在进行的任务完成。

可以使用下面的代码片段来创建一个`Future`的实例:

```java
public void invoke() {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
 
    Future<String> future = executorService.submit(() -> {
        // ...
        Thread.sleep(10000l);
        return "Hello world";
    });
}
```

以下代码片段来检查`Future`的结果是否准备好，并在计算完成后获取数据:

```java
if (future.isDone() && !future.isCancelled()) {
    try {
        str = future.get();
    } catch (InterruptedException | ExecutionException e) {
        e.printStackTrace();
    }
}
```

可以为给定的操作指定超时。如果任务花费的时间超过这个时间，则抛出`TimeoutException`:

```java
try {
    future.get(10, TimeUnit.SECONDS);
} catch (InterruptedException | ExecutionException | TimeoutException e) {
    e.printStackTrace();
}
```

## CountDownLatch

`CountDownLatch`在`JDK 5`中引入，是一个实用程序类，它阻塞一组线程，直到某个操作完成。

`CountDownLatch`用计数器(整数类型)初始化，这个计数器在从属线程完成执行时递减。但是一旦计数器达到0其他线程就会被释放。

## CyclicBarrier

`CyclicBarrier`的工作原理和`CountDownLatch`几乎一样，只是我们可以重复使用它。与`CountDownLatch`不同，它允许多个线程在调用最后一个任务之前使用`await()`方法彼此等待。

创建一个可运行的任务实例初始化CyclicBarrier:

```java
public class Task implements Runnable {
 
    private CyclicBarrier barrier;
 
    public Task(CyclicBarrier barrier) {
        this.barrier = barrier;
    }
 
    @Override
    public void run() {
        try {
            LOG.info(Thread.currentThread().getName() + 
              " is waiting");
            barrier.await();
            LOG.info(Thread.currentThread().getName() + 
              " is released");
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
 
}
```

调用一些线程来竞争CyclicBarrier:

```java
public void start() {
 
    CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> {
        // ...
        LOG.info("All previous tasks are completed");
    });
 
    Thread t1 = new Thread(new Task(cyclicBarrier), "T1"); 
    Thread t2 = new Thread(new Task(cyclicBarrier), "T2"); 
    Thread t3 = new Thread(new Task(cyclicBarrier), "T3"); 
 
    if (!cyclicBarrier.isBroken()) { 
        t1.start(); 
        t2.start(); 
        t3.start(); 
    }
}
```

`isBroken()`方法检查在执行期间是否有线程被中断。我们应该总是在执行实际的过程之前执行这个检查。

## ThreadFactory

顾名思义，`ThreadFactory`充当线程池，根据需要创建新线程。它消除了为实现高效的线程创建机制而编写大量样板代码的需要。

```java
public class MyThreadFactory implements ThreadFactory {
    private int threadId;
    private String name;

    public MyThreadFactory(String name) {
        threadId = 1;
        this.name = name;
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r, name + "-Thread_" + threadId);
        LOG.info("created new thread with id : " + threadId +
            " and name : " + t.getName());
        threadId++;
        return t;
    }
}
```

使用`newThread(Runnable r)`方法在运行时创建一个新线程:

```java
MyThreadFactory factory = new MyThreadFactory( 
    "MyThreadFactory");
for (int i = 0; i < 10; i++) { 
    Thread t = factory.newThread(new Task());
    t.start(); 
}
```

## BlockingQueue

在异步编程中，最常见的模式之一是生产者-消费者模式。`java.util.concurrent`中`BlockingQueue`在这些异步场景中非常有用。

## DelayQueue

`DelayQueue`是一个无限大小的元素阻塞队列，只有当元素的过期时，才能将其移出。因此，最上面的元素(head)将有最多的延迟，它将被轮询到最后。

## Locks

`Lock`是一个实用工具，用于阻止其他线程访问特定的代码段。

`Synchronized`和`Lock`之间的主要区别是`Synchronized`完全包含在方法中;但是，我们可以在不同的方法中使用`Lock` API的`Lock()`和`unlock()`操作。

## 总结

在这篇高级的概述性文章中，我们重点讨论了`java.util.concurrent`的不同实用程序。
