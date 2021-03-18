---
title: Java线程安全的集合
author: lijiabao
date: 2021-03-15 14:00
categories:  Java并发
tags: Java并发

---

主要介绍一下java线程的基本状态以及属性。同时介绍了Java线程的竞争条件以及处理竞争条件的集中方法，也就是达成同步的集中方法。

## 线程安全的集合

如果多个线程要并发的修改一个数据结构，很容易破坏这个数据结构。如果一个正在修改数据结构的线程突然被中断，这时候另外一个线程操作这个数据结构，可能会导致抛出异常或者无线循环。下面将会介绍一下线程安全的一些Java集合

### 阻塞队列

很多的线程问题可以用一个或者多个队列以优雅安全的方式描述。生产者线程向队列插入元素，消费者线程获取元素。使用队列可以安全的从一个线程向另外一个线程传递数据

当试图从一个队列获取元素而队列为空或者试图向队列插入元素而队列已满的时，阻塞队列将会导致线程的阻塞。阻塞队列在协调多个线程的合作时，是个很有用的工具工作线程可以周期性的将中间结果存储在阻塞队列时，其他工作线程移除中间结果，并进一步进行修改。

队列会自动的平衡负载。如果第一组线程运行的比第二组慢，当第二组在等待结果时会阻塞。如果运行的块，队列填满，会导致阻塞，直到较慢的线程追赶上来。

**下面是阻塞队列的方法：**

|  方法   |          操作          |               意外时操作                |
| :-----: | :--------------------: | :-------------------------------------: |
|   add   |      添加一个元素      | 队列满，抛出`IllegalStateException`异常 |
| element |      返回队头元素      |  队列空，抛出`NoSuchElementException`   |
|  peek   | 添加一个元素并返回true |            队列空，返回null             |
|   put   |      添加一个元素      |            队列满，则被阻塞             |
|  poll   |   移除并返回队头元素   |            队列空，返回null             |
| remove  |   移除并返回队头元素   |  队列空，抛出`NoSuchElementException`   |
|  take   |   移除并返回队头元素   |            队列空，则被阻塞             |
|  offer  | 添加一个元素并返回true |            队列满，返回false            |

阻塞队列方法主要分为三类：

- 线程管理：使用take和put方法
- 会报错的一类：试图向满队列插入元素或者从空队列获取元素，如add，element和remove
- 多线程程序中，可能随时变满变空，应使用offer、poll和peek。offer个poll方法都有超时时间的重载方法，如果超时返回null

`java.util.concurrent`包提供了阻塞队列的几个变体。

- `LinkedBlockingQueue`:容量没有上限，也可以指定最大容量。是一个双端队列
- `ArrayBlockingQueue`:需要制定容量，还有一个可选参数指定是否需要公平性，设置了公平参数会优先处理等待时间长的线程，设置了公平参数会影响性能，只有确实是十分需要时才使用
- `PriorityBlockingQueue`:是优先队列而不是先进先出队列，元素按照他们的优先级顺序移除。这个队列没有容量上限，但是如果队列是空，获取元素操作会阻塞
- `DelayQueue`包含实现了`Delayed`接口的对象.其中的`getDelay`方法返回对象的剩余延迟.负值表示延迟已经结束.元素只有在延迟结束的情况才能从队列删除.还需要实现`compareTo`方法,使用这个方法对元素排序
  - Java 7新增了一个`TransferQueue`接口,允许生产者线程等待,直到消费者就绪可以接收元素.如果生产者调`q.transfer(item)`.这个调用会阻塞,直到另外一个线程将元素item删除.`LinkedBlockingQueue`实现了这个接口

#### `java.util.concurrent`包下面的一些和队列相关的类和方法

**`java.util.concurrent.ArrayBlockingQueue<E>`类**

- `ArrayBlockingQueue(int capacity)`指定容量的构造器
- `ArrayBlockingQueue(int capacity, boolean fair)`指定容量和公平策略的构造器

**`java.util.concurrent.LinkedBlockingQueue<E>`和`java.util.concurrent.LinkedBlockingDeque<E>`类**

- `LinkedBlockingQueue()`和`LinkedBlockingDeque()`构造一个无上限的阻塞队列或者双向队列，实现为一个链表
- `LinkedBlockingQueue(int capacity)`和`LinkedBlockingDeque(int capacity)`构造为一个有限容量的

**`java.util.concurrent.DelayQueue<E extends Delayed>`类**

- `DelayQueue()`构建一个包含delayed元素的无上限阻塞队列,只有那些延迟已经到期的元素可以从队列中移除

**`java.util.concurrent.Delayed`接口**

- `long getDelay(TimeUnit unit)`得到该对象的延迟,用给定的单位度量

`java.util.concurrent.PriorityBlockingQueue<E>`类:

- `PriorityBlockingQueue()`
- `PriorityBlockingQueue(int capacity)`
- `PriorityBlockingQueue(int capacity, Comparetor<? super E> conparetor)`构造阻塞优先队列,实现为一个堆.优先队列的默认初始容量为11.如果没有指定比较器,元素必须实现Comparable接口

`java.util.concurrent.BlockingQueue<E>`类:

- `void put(E element)`:添加元素,在必要时阻塞
- `E take()`:移除并返回队头元素,必要时阻塞
- `boolean offer(E element, long time, TimeUnit unit)`添加给定的元素,必要时阻塞,直到元素已经添加或者超时
- `E poll(long time, TimeUnit unit)`移除并返回队头元素,必要时阻塞,直到元素可用或者超时用完.失败返回null

`java.util.concurrent.BlockingDeque<E>`类:

- `void putFirst(E element)`和`void putLast(E element)`添加元素,必要时阻塞
- `E takeFirst()`和`E takeLast()`移除并返回队头或者队尾元素必要时阻塞
- `boolean offerFirst(E element, long time, TimeUnit unit)`和`boolean offerLast(E element, long time, TimeUnit unit)`添加给定的元素,成功返回true,必要时阻塞,直到超时或者已经添加
- `E pollFirst(long time, TimeUnit unit)`和`E pollLast(long time, TimeUnit unit)`移除并返回队头元素,必要时阻塞,直到元素可用或者超时,失败时返回null

`java.util.concurrent.TransferQueue<E>`接口

- `void transfer(E element)`
- `boolean tryTransfer(E element, long time, TimeUnit unit)`传输一个值,或者尝试在给定的时间内传输这个值,这个调用将阻塞,直到另外以恶线程将元素删除,第二个方法会在调用成功时返回true

### 高效的映射、集和队列



### 映射条目的原子更新



