---
title: Java任务和线程池
author: lijiabao
date: 2021-03-15 16:00

categories:  Java并发
tags: Java并发
---

主要介绍一下Java如何通过任务和线程池来减少资源开销。

## 任务和线程池

构造一个新的线程开销有些大，因为这涉及到与操作系统的交互。如果你的程序中创建了大量的声明周期短的线程，那么不应该把每个任务映射到一个单独的线程，而应该使用线程池。线程池中有包含多个准备运行的线程，为线程池提供一个Runnable，就会有一个线程调用run方法，当run方法 退出，这个线程不会死亡，而留在线程池中为下一个请求提供服务。

### Callable和Future

Runnable封装的是一个异步运行的任务，你可以把他想象成为一个没有参数和返回值的一部方法。Callable和Runnable类似，但是有返回值。Callable是一个参数化的类型，只有一个方法call

执行callable的一种方法是使用FutureTask，它实现了Future和Runnable接口，运行这个任务的代码：

```java
Callable<Integer> task = ...;
var futureTask = new FutureTask<Integer>(task);
var t = new Thread(futureTask);
t.start();
...
Integer result = task.get();
// 还有一种更加常见的情况，将Callable传入到一个执行器
```

`java.util.concurrent.Callable<V>`接口：

- `V call()`运行一个将产生结果的任务

`java.util.concurrent.Future<V>`接口

- `V get()`和`V get(long time, TimeUnit unit)`获取结果，这个方法会阻塞，直到结果可用或者超过了指定的时间。如果不成功，第二个方法会抛出`TimeoutException`异常
- `boolean cancel(boolean mayInterrupt)`尝试取消这个任务，如果任务开始了，并且mayInterrupt参数为true，他就会被中断，如果成功执行了取消操作，则返回true
- `boolean isCancelled()`如果任务在完成前被取消，返回true
- `boolean isDone()`如果任务结束无论是正常完成、中途取消还是发生异常都返回true

`java.util.concurrent.FutureTask<V>`类

- `FutureTask(Callable<V task)`
- `FutureTask(Runnable task, V result)`构建一个既是Future<V>又是Runnable的对象

### 执行器`Executor`

执行器类有许多的静态工厂方法，用来构建线程池：(返回值是`ExecutorService`)

|               方法               |                             描述                             |
| :------------------------------: | :----------------------------------------------------------: |
|      `newCachedThreadPool`       | 必要时（有用现成的，没有再创建）创建新线程，空闲线程保留60s。线程池创建好之后立即执行各个任务 |
|       `newFixedThreadPool`       | 池中包含固定数目的线程，空闲线程一直保留。如果提交的任务多于空闲线程，未分配线程会放到队列中，等待其他任务完成任务再执行 |
|      `newWorkStealingPool`       | 一种适合fork-join任务的线程池，其中复杂任务会分解为更简单的任务。其中复杂的任务会分解为更简单的任务，空闲线程会密取较简单的任务 |
|    `newSingleThreadExecutor`     |           只有一个线程的池，会顺序的执行提交的任务           |
|     `newSceduledThreadPool`      | 用于调度执行的固定线程池,返回值是`ScheduledExecutorService`  |
| `newSingleThreadSceduledExector` |  用于调度执行的单线程池,返回值是`ScheduledExecutorService`   |

使用执行器的步骤：

1. 调用执行器类的静态方法创建线程池
2. 调用submit方法提交Runnable或Callable对象
3. 保存好返回的Future对象以便获取结果或者取消任务
4. 当不想再 提交任何任务时调用shutdown

`ScheduledExecutorService`接口为调度执行或重复执行任务提供了一些方法,这是对支持建立线程池的`java.util.Timer`的泛化.其中Executor类的`newScheduledThread Poll`和`newSingleThreadScheduledExecutor`方法返回实现`ScheduledExecutorService`接口的对象.

`java.util.concurrent.ScheduledExecutorService`接口:

- `ScheduledFuture<V>(Callable<V> task, long time, TImeUnit unit)`
- `ScheduledFuture<V>(Runnable task, long time, TImeUnit unit)`调度在指定时间之后执行任务
- `ScheduledFuture<?> ScheduleAtFixedRate(Runnable task, long initialDelay, long period, TimeUnit unit)`调度在初始延迟后,周期性的运行给定的任务,周期长度时period
- `ScheduledFuture<?> ScheduleWithFixedDelay(Runnable task, long initialDelay, long delay, TimeUnit unit)`调度在初始延迟后,周期性的运行给定的任务,周期长度时period.在一次调用完成和下一次调用开始之间有delay的延迟

`java.util.concurrent.ExecutorService`对象:

- `Future<T> submit(Callable<T> task)`
- `Future<T> submit(Runnable task, T result)`
- `Future<T> submit(Runnable task)`提交指定的任务来执行
- `void shutdown()`关闭服务,完成已经提交的任务但不接受新的任务

`java.util.concurrent.ThreadPollExecutor`对象

- `int getLargestPollSize()`返回该执行器生命周期内的线程池的最大大小

### 控制任务组

有时候使用执行器有更策略性的原因:需要控制一组相关的任务.例如可以在执行器中使用shundown方法取消所有的任务.

`invokeAll()`方法提交一个Callable对象集合中的所有对象,并返回某个已完成任务结果.这并不知道完成的任务是哪一个,往往是最快完成的.如果愿意接受任何一个结果,可以使用这个方法.或者是进行多个任务计算,但是只要有一个计算得到结果,就可以结束运行了.

```java
List<Callable<T>> tasks = ...;
List<Future<T>> results = executor.invokeAll(tasks);
for (Future<T> result: results){
    result.get();
}
// 第一个result.get()会阻塞直到第一个结果可用.
```

虽然上述步骤没啥问题,但是我们还是希望按照计算出结果的顺序得到这些结果,可以利用`ExecutorCompletionService`来管理

```java
var service = new ExecutorCompletionService<T>(executor);
for (Callable<T> task: tasks){service.submit(task);}
for (int i=0;i<tasks.size();i++){service.take().get();}
```

`java.util.concurrent.ExecutorService`类:

- `T invokeAny(Collection<Callable<T>> tasks)`
- `T invokeAny(Collection<Callable<T>> tasks, long timeout, TimeUnit unit)`执行给定的任务,返回其中一个任务的结果,如果超时,第二个方法抛出`TimeoutException`异常
- `List<Future<T>> invokeAll(Collection<Callable<T>> tasks)`
- `List<Future<T>> invokeAll(Collection<Callable<T>> tasks, long timeout, TimeUnit unit)`执行给定的任务,返回所有任务的结果,如果超时,第二个方法抛出`TimeoutException`异常

`java.util.concurrent.CompletionService<V>`类

- `ExecutorCompletionService(Executor e)`构造一个执行器完成服务来收集给定执行器的结果
- `Future<V> submit(Callable<V> task)`
- `Future<V> submit(Runnable task, V result)`提交一个任务给底层的执行器
- `Future<V> take()`移除下一个已经完成的结果,如果没有可用的已完成结果,阻塞
- `Future<V> poll()`
- `Future<V> poll(long time, TimeUnit unit)`移除并返回下一个已经完成的结果,如果没有可用的已完成的结果,返回null.第二个方法会等待指定的时间

### fork-join框架

有些应用可能对每个处理器内核分别使用一个线程,用来完成计算密集型任务,如图像处理和视频处理.Java 7引入fork-join框架用来支持这一类应用操作.假设有一个处理任务,它可以分解成很多个子任务.

```java
// 类似下面的这种例子
if (problemSize < threshold){
	直接解决问题;
}
else{
    将问题分解为多个子任务递归处理;
    整合所有子任务的结果;
}
```

要采用框架可用的一种方式完成递归计算,需要提供一个扩了`RecursiveTask<T>`的类或者提供一个扩展`RecursiveAction`的类(如果不生成任何结果)然后再覆盖compute方法来生成并调用子任务,然后合并结果.

```java
class Counter extends RecursiveTask<Integer>{
    if (to - from < THRESHOLD){
        直接计算解决问题;
    }
    else{
        int mid = (from + to) / 2;
        var first = new Counter(values, from, mid, filter);
        var second = new Counter(values, mid, to, filter);
        invokeAll(first, second);
        return first.join() + second.join();
    }
}
// invokeAll接受很多任务并阻塞,直到所有任务全部完成,join方法将生成结果.
// 对每个子任务应用join,并返回其总和
```

fork-join框架使用了一种有效的智能方法来平衡可用线程的工作负载,这种方法称为工作密取(work stealing).每个工作线程都有一个双端队列来完成任务,一个工作线程将子任务压入双端队列的队头(只有一个线程可以访问队头,不需要加锁)一个工作线程空闲时,会从另一个双端队列的队尾密取一个任务.大的子任务都在队尾,这种密取很少出现.

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;
import java.util.concurrent.RecursiveTask;
import java.util.function.DoublePredicate;
import java.util.function.Predicate;

/**
 * 这个类主要是用来展示怎么使用fork-join框架来实现大的人物分割成多个小任务
 * 然后再去递归的计算小人物，最后整合所有的小任务的结果，可以达到很好的并发的处理计算密集型任务
 */
public class ForkjoinTest {
    public static void main(String[] args) {
        final int SIZE = 10000000;
        var numbers = new double[SIZE];
        for (int i = 0; i < SIZE; i++) numbers[i] = Math.random();
        var counter = new Counter(numbers, 0, numbers.length, x -> x > 0.5);
        var pool = new ForkJoinPool();
        pool.invoke(counter);
        System.out.println("最终的结果是有" + counter.join() + "个数大于0.5");

    }
}

class Counter extends RecursiveTask<Integer>{
    public static final int THRESHOLD = 1000;
    private double[] values;
    private int from;
    private int to;
    private DoublePredicate filter;

    /**
     * fork-join框架扩展了RecursiveTask类，用于简单的计算满足某个条件的数有多少个
     * @param values 这是给定的数组集合，需要用来判断的数据存放在这里作为参数传递给构造器
     * @param from 起始索引
     * @param to 结束索引
     * @param filter 过滤器，用于判断满足是否满足某个条件
     */
    public Counter(double[] values, int from, int to, DoublePredicate filter)
    {
        this.values = values;
        this.from = from;
        this.to = to;
        this.filter = filter;
    }

    /**
     * 对于扩展了RecursiveTask和RecuesiveAction的类，需要重载这个方法
     * @return 计算操作的结果
     */
    @Override
    protected Integer compute() {
        if (to -from < THRESHOLD){
            int count = 0;
            for (int i = from; i < to; i++){
                if (filter.test(values[i])) count++;
            }
            return count;
        }
        else {
            int mid = (from + to) / 2;
            var first = new Counter(values, from, mid, filter);
            var second = new Counter(values, mid, to, filter);
            invokeAll(first, second);
            return first.join() + second.join();
        }
    }
}
```

！！！fork-join池是针对非阻塞工作负载优化的,如果向一个fork-join池增加很多阻塞任务,会让它无法有效工作