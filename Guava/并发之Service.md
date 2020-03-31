Service

## 概述

Guava包里的[Service](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html)用于封装一个服务对象的运行状态、包括*start*和*stop*等方法。例如web服务器，RPC服务器、计时器等可以实现这个接口。对此类服务的状态管理并不轻松、需要对服务的开启/关闭进行妥善管理、特别是在多线程环境下尤为复杂。Guava 包提供了一些基础类帮助你管理复杂的状态转换逻辑和同步细节。

## 使用一个服务

一个服务正常生命周期有：

* [Service.State.NEW](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#NEW)
* [Service.State.STARTING](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#STARTING)
* [Service.State.RUNNING](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#RUNNING)
* [Service.State.STOPPING](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#STOPPING)
* [Service.State.TERMINATED](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#TERMINATED)

服务一旦被停止就无法再重新启动了。如果服务在*starting、running、stopping*状态出现问题、会进入[Service.State.FAILED](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#FAILED)状态。

调用 [startAsync()]()方法可以异步开启一个服务,同时返回this对象形成方法调用链。注意：只有在当前服务的状态是NEW时才能调用`startAsync()`方法，因此最好在应用中有一个统一的地方初始化相关服务。停止一个服务也是类似的、使用异步方法 [stopAsync()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#stopAsync--) 。但是不像`startAsync()`,多次调用这个方法是安全的。这是为了方便处理关闭服务时候的锁竞争问题。

Service 也提供了一些方法用于等待服务状态转换的完成:

* 通过 [addListener()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#addListener--)方法异步添加监听器。此方法允许你添加一个`Service.Listener`、它会在每次服务状态转换的时候被调用。注意：最好在服务启动之前添加`Listener`（这时的状态是 NEW）、否则之前已发生的状态转换事件是无法在新添加的`Listener`上被重新触发的。

* 同步使用 [awaitRunning()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#awaitRunning--)。这个方法不能被打断、不强制捕获异常、一旦服务启动就会返回。如果服务没有成功启动，会抛出 IllegalStateException 异常。同样的， [awaitTerminated()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#awaitTerminated--) 方法会等待服务达到终止状态（TERMINATED 或者 FAILED）。两个方法都有重载方法允许传入超时时间。

`Service`接口本身实现起来会比较复杂、且容易碰到一些捉摸不透的问题。因此我们不推荐直接实现这个接口。而是请继承 Guava 包里已经封装好的基础抽象类。每个基础类支持一种特定的线程模型。

## 实现类

### AbstractIdleService

[AbstractIdleService](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html)类简单实现了`Service`接口、其在*running*状态时不会执行任何动作–因此在*running*时也不需要启动线程–但需要处理开启/关闭动作。要实现一个此类的服务，只需继承`AbstractIdleService`类，然后自己实现 [startUp()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html#startUp--) 和 [shutDown()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html#shutDown--)方法就可以了。

```java
protected void startUp() {
  servlets.add(new GcStatsServlet());
}
protected void shutDown() {}
```

如上面的例子、由于任何请求到`GcStatsServlet`时已经会有现成线程处理了，所以在服务运行时就不需要做什么额外动作了。

### AbstractExecutionThreadService

[AbstractExecutionThreadService](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/AbstractExecutionThreadService.html)通过单线程处理启动、运行、和关闭等操作。你必须重载`run()`方法，同时需要能响应停止服务的请求。具体的实现可以在一个循环内做处理：

```java
public void run() {
  while (isRunning()) {
    // perform a unit of work
  }
}
```

另外，你还可以重载任一方法让`run()`方法结束返回。

重载`startUp()`和`shutDown()`方法是可选的，不影响服务本身状态的管理

```java
protected void startUp() {
  dispatcher.listenForConnections(port, queue);
}
protected void run() {
  Connection connection;
  while ((connection = queue.take() != POISON)) {
    process(connection);
  }
}
protected void triggerShutdown() {
  dispatcher.stopListeningForConnections(queue);
  queue.put(POISON);
}
```

`start()`内部会调用`startUp()`方法，创建一个线程、然后在线程内调用`run()`方法。`stop()`会调用`triggerShutdown()`方法并且等待线程终止。

### AbstractScheduledService

[AbstractScheduledService](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html)类用于在运行时处理一些周期性的任务。子类可以实现 [runOneIteration()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html#runOneIteration--)方法定义一个周期执行的任务，以及相应的`startUp()`和`shutDown()`方法。

为了能够描述执行周期，你需要实现 [scheduler()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html#scheduler--)方法。通常情况下，你可以使用[AbstractScheduledService.Scheduler](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html)类提供的两种调度器：[newFixedRateSchedule(initialDelay, delay, TimeUnit)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html#newFixedRateSchedule-long-long-java.util.concurrent.TimeUnit-)和[newFixedDelaySchedule(initialDelay, delay, TimeUnit)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html#newFixedDelaySchedule-long-long-java.util.concurrent.TimeUnit-)，类似于JDK并发包中`ScheduledExecutorService` 类提供的两种调度方式。如要自定义schedules 则可以使用[CustomScheduler](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.CustomScheduler.html)类来辅助实现；具体用法见 javadoc。

### AbstractService

如需要自定义的线程管理、可以通过扩展[AbstractService](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html)类来实现。一般情况下、使用上面的几个实现类就已经满足需求了，但如果在服务执行过程中有一些特定的线程处理需求、则建议继承`AbstractService` 类。

继承`AbstractService`方法必须实现两个方法.

* [doStart()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#doStart--): 首次调用 startAsync()时会同时调用`doStart()`,`doStart()`内部需要处理所有的初始化工作、如果启动成功则调用 [notifyStarted()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyStarted--)方法；启动失败则调用[notifyFailed()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyFailed-java.lang.Throwable-)。
* [doStop()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#doStop--): 首次调用`stopAsync()`会同时调用`doStop()`,`doStop()`要做的事情就是停止服务，如果停止成功则调用 [notifyStopped()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyStopped-)方法；停止失败则调用[notifyFailed()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyFailed-java.lang.Throwable-)方法。

`doStart`和 `doStop`方法的实现需要考虑下性能，尽可能的低延迟。如果初始化的开销较大，如读文件，打开网络连接，或者其他任何可能引起阻塞的操作，建议移到另外一个单独的线程去处理。

## ServiceManager

除了对`Service`接口提供基础的实现类，Guava还提供了[ServiceManager](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html)类使得涉及到多个`Service`集合的操作更加容易。通过实例化 `ServiceManager`类来创建一个`Service`集合，你可以通过以下方法来管理它们：

* [startAsync()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#startAsync--) ： 将启动所有被管理的服务。如果当前服务的状态都是 NEW 的话、那么你只能调用该方法一次、这跟`Service#startAsync()`是一样的。
* [stopAsync()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#stopAsync--) ：将停止所有被管理的服务。
addListener ： 会添加一个`ServiceManager.Listener`，在服务状态转换中会调用该`Listener`
* [awaitHealthy()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#awaitHealthy--) ：会等待所有的服务达到`Running` 状态。
* [awaitStopped()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#awaitStopped--)：会等待所有服务达到终止状态。

检测类的方法有：

* [isHealthy()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#isHealthy--) ：如果所有的服务处于`Running`状态、会返回`True`。
* [servicesByState()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#servicesByState--)：以状态为索引返回当前所有服务的快照。
* [startupTimes()](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#startupTimes--) ：返回一个`Map`对象，记录被管理的服务启动的耗时、以毫秒为单位，同时`Map`默认按启动时间排序。

我们建议整个服务的生命周期都能通过`ServiceManager`来管理，不过即使状态转换是通过其他机制触发的、也不影响`ServiceManager`方法的正确执行。例如：当一个服务不是通过`startAsync()`、而是其他机制启动时，`listeners`仍然可以被正常调用、`awaitHealthy()`也能够正常工作。`ServiceManager`唯一强制的要求是当其被创建时所有的服务必须处于`New`状态。
