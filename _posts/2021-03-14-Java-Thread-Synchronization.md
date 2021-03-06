---
title: Java线程同步
author: lijiabao
date: 2021-03-14 13:00
categories:  Java并发
tags: Java并发
---

在这里主要介绍一下java线程的基本状态以及属性。同时介绍了Java线程的竞争条件以及处理竞争条件的集中方法，也就是达成同步的集中方法。

## Java线程

### 线程的状态

线程主要可以有以下的6种状态

- New(新建)当使用`new Thread()`时,这个线程还没开始运行,属于新建状态
- Runnable(可运行),一旦调用了`start()`方法后,线程就处于可运行状态,可运行的线程可能在运行,也可能不在运行
- Blocked(阻塞)处于阻塞状态的线程,暂时是不活动的,不运行任何代码.消耗最少的资源.需要线程调度器重新激活这个线程.当线程试图获取锁时,如果这个锁被其他线程占用,这个线程就会被阻塞,只有当其他线程释放了这个锁,且线程调度器允许线程使用,才会变成非阻塞状态
- Waiting(等待)等待状态,和阻塞状态是类似.当线程等待另外一个线程通知调度器达到某个条件时,进入等待状态,直到条件达到,调度去通知这个线程,才会变成非等待状态.调用`Object.wait()`或`Thread.join()`方法或等待`java.util.concurrent`库中的Lock或者Condition时,会出现这种状态
- Time Waiting(计时等待)和上述的阻塞和等待类似,只是有个超时时间,期满或者收到通知就会结束等待状态.当调用有超时参数的方法时,会进入计时等待的状态,这种状态会保持到超时期满或者接收到适当的通知.超时参数的方法有`Thread.sleep()`有计时参数的`Object.wait()``Thread.join()``Lock.tryLock``Condition.wait()`
- Terminated(停止)当线程run完毕之后就处于这个状态,或者没有捕获的异常终止了这个方法

![线程状态](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E7%BA%BF%E7%A8%8B%E7%8A%B6%E6%80%81.png)

**Thread类的几个方法:**

- `void join()`等待终止指定的线程
- `void join(long millis)`等待指定的线程终止或者等待经过指定的毫秒数
- `Thread State getState()`得到这个线程的状态;状态值有`NEW、RUNNABLE、BLOCKED、WATTING、TIMED_WAITING和TERMINATED`

### 线程属性

#### 中断线程

当run方法体中最后一条语句执行完后再执行return语句返回时，或者出现了异常，线程终止。早期的java有一个stop方法终止线程，但是已经弃用

除了stop方法，没有方法可以强制线程终止，但是`interrupt`方法可以请求终止一个线程。当对一个线程调用interrupt方法时，就会设置线程的中断状态，每个线程都有这个标志。每个线程应该时不时检查一下判断线程是否被中断。

```java
Runable run = () -> {
    try
    {
        // 如果线程被阻塞，无法查看中断状态，因此需要捕获是否出现Iterrupted异常
        while (!Thread.cunrrentThread().isInterrupted() && 其他判断){
           	do some work;
        }
    }catch (InterruptedException e){
        // 线程由于阻塞而被中断
    }
    finally{
        // 执行一些必要的清理工作
    }
    // 执行到run的最后语句，线程终止
}
```

如果每次工作迭代都调用了sleep方法或者其他可中断方法，检查Interrupted就没必要了，实际如果使用了sleep并不会休眠，只会清除中断状态，并抛出异常，因此，sleep方法时，需要捕获InterruptedException异常

对于抑制Interrupted异常的底层代码如：

```java
void func() {
    try{
        sleep(delay);
    }catch (InterruptedException e){
        // do nothing
    }
}
```

不要这样做：如果不知到捕获了异常做什么，可以在方法中抛出。对于上面的情况，有两种解决：

- catch语句中调用`Thread.currentThread().interrupt()`设置中断状态，这样调用者就可以检查状态
- 在方法体外面抛出这个异常，去掉try语句块

**和中断状态相关的线程类的方法：**

- `void interrupt()`向线程发送中断请求,线程的中断状态将设置为true,如果sleep调用阻塞当前线程,将抛出异常
- `static boolean interrupted()`测试当前线程是否中断.调用了这个方法之后,会将当前中断设为fasle
- `boolean isIterrupted()`测试线程是否被中断,这个调用不改变线程状态
- `static Thread currentThread()`获取当前正在执行线程的Thread对象

#### 守护线程

调用`setDaemon(true)`将一个线程设置为守护线程.守护线程的用途是为其他线程提供服务.计时器就是一个例子,定时发送计时器滴答信息给其他线程,清空过时缓存的线程也是守护线程.当只剩下守护线程时,,虚拟机就会退出,因为没有必要再继续运行程序了.下面是Thread类的设置守护线程的方法

- `void setDaemon()`标识该线程为守护线程或用户线程,只能在线程启动之前调用这个方法

#### 线程名

默认情况下,线程的名字是`Thread-1`之类的,可以使用`setName()`为线程设置任何名字,也可以在线程创建时,设置name参数值`setName()`在线程转储时很有用

#### 未捕获异常的处理器

线程的run方法不可抛出任何检查型的异常,但是非检查型的异常可能会导致线程终止,线程因此会死亡.但是实际上线程死亡之前,异常会传递到一个用于处理未捕获异常的处理器.这个处理器必须属于一个实现了`Thread.UncaughtExceptionHandler`接口的类,这个接口只有一个方法

- `void uncaughtException(Thread t, Throwable e)`

可以使用`setUncaughtExceptionHandler`方法为任何线程安装一个处理器,也可以使用thread类的静态方法`setDefaultUncaughtExceptionHandler`为所有的线程安装一个默认的处理器.处理器可以使用日志API将未捕获异常的报告发送到一个日志文件.

如果没有安装默认处理器,默认处理器为null.但是如果没有单个线程安装处理器,那么处理器就是该线程的ThreadGroup对象

> 线程组是一个可以一起管理的线程的集合.默认情况,创建的所有线程都属于同一个线程组,但是也可以建立其他的组.但是由于现在有更好的特性来处理器线程集合,建议不要在自己的程序中使用线程组

**ThreadGroup类实现了Thread.UncaughtExceptionHandler接口.他的uncaughtException执行下述的操作:**

1. 如果该线程有父线程组,调用父线程组的uncaughtException方法
2. 否则,如果`Thread.getDefaultExceptionHandler`返回了非null的处理器,调用这个处理器
3. 否则,如果Throwable是ThreadDeath的实例什么都不做
4. 否则,将线程的名字以及Throwable的栈轨迹输出到System.err

**线程类的方法:**

- `static voidsetDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler handler)`设置未捕获异常的默认处理器
- `static Thread.UncaughtExceptionHandler getDefaultUncaughtExceptionHandler()`获取默认的未捕获异常处理器
- `void setUncaughtExceptionHandler(Thread.UncaughtExceptionHandler handler)`设置未捕获异常处理器
- `Thread.UncaughtExceptionHandler getUncaughtExceptionHandler()`获取未捕获异常处理器

#### 线程优先级

Java中,每个线程都有一个优先级.默认情况下,线程会继承构建他的那个线程的优先级.可以使用`setPriority`方法提高或者降低线程的优先级.

优先级设置的范围是`MIN_PRIORITY`1~`MAX_PRIORITY`10之间的任何数.`NORMAL_PRIORITY`定义成5,也是java的默认优先级

当线程调度器选择新线程时,会优先选择较高优先级的线程.但是线程的优先级高度依赖于系统,在不同的系统平台下,Java的线程优先级会发生改变.Java线程的优先级会映射到宿主机系统的优先级

早期Java版本并没有使用操作系统的线程优先级,因此优先级设置会比较有用,但是使用了操作系统的线程优先级之后,不要使用优先级了



### 资源同步

当两个线程同时竞争一个资源的时候,这时候就会出现问题.可能两个线程的数据会相互覆盖,导致数据出错对象破坏之类的问题,这种情况被称之为竞态条件(race condition)。出现竞态条件的原因可能时某一线程执行的时候，另外一个线程中断了这个线程，从而出现多个线程共同抢夺一个资源的问题。因此，为了解决这个问题，需要在线程失去控制之前确保方法已经执行完毕

#### 锁对象

Java中有两种机制来防止并发访问代码块。java语言提供了synchronized关键字来达到这一目的，另外Java5引入了`ReentrantLock`类.synchronized关键字会自动提供一个锁以及相关的条件,对于大多数需要显式锁的情况,这种机制很强大,也很便利

**ReentrantLock保护代码块的基本结构:**

```java
// 首先需要定义一个ReentrantLock对象
private Lock myLock = new java.util.concurrent.locks.ReentrantLock();
myLock.lock(); //一个ReentrantLock对象
try{
    关键代码块
}
finally
{
    myLock.unlock(); // 即使抛出了异常,也能确保锁被打开
}
```

这个结构可以确保任何时刻都只有一个线程进入try语句块.一旦一个线程锁定了锁对象,其他任何线程都无法通过lock语句,当某个线程调用lock时,其他的线程会暂停,直到这个线程释放这个锁对象!!!使用锁时,不可以使用try-with-resource语句。

如果两个线程同时访问同一个对象，这个对象有自己的重入锁，那么，锁可以保证串行化访问，但是访问不同的对象的时候，由于锁不同，并不会阻塞线程。<font color='red'>**线程可以反复拥有已经拥有过的锁**</font>，但要确保线程每一次调用锁之后都使用unlock释放。因此被一个锁保护的代码块可以调用另外使用相同锁的方法。

需要注意！！！临界区的代码不要因为抛出异常而跳出临界区，这样可能导致虽然释放了锁，但对象处于破坏状态

类`java.util.concurrent.locks.Lock`:

- `void lock()`获得这个锁:如果锁被其他线程占有,则阻塞
- `void unlock`释放这个锁

类`java.util.concurrent.locks.ReentrantLock`:

- `ReentrantLock()`构造一个重入锁,可以用来保护临界区(critical section)
- `ReentrantLock(boolean fair)`构造一个采用公平策略的锁。一个公平锁倾向于等待时间最长的锁，但是这种公平保证严重影响性能，默认情况不要求锁是公平

#### 条件对象

通常，线程进入临界区后却发现只有满足了某个条件之后他才执行，因此可以使用一个条件对象来管理那些已经获得了锁却不能执行有效工作的线程。

使用条件对象的情况是：在你需要满足某些条件的情况下，你才可以执行这个线程的任务。但是如果你将条件判断放在获取锁的代码块之外的时候，还是可能会被中断。因此需要把条件判断也置于锁内才行。

一个锁对象可以拥有多个相关联的条件对象。可以使用newCondition方法获取一个条件对象，习惯上会给条件对象一个可以反映条件的名字。如果发现条件不满足，会调用条件对象的等待wait方法。当进入等待时，这个线程会暂停并放弃锁。当线程调用wait方法时，他就进入线程的等待集合中。此时如果锁可用时，并不会变为可运行状态，实际它仍然是非活动状态，只有当某个线程在同一条件上调用了signalAll方法，具体的简单实现的代码如下：

```java
public void transfer(){
    try{
        while (!condition){
            conditionInstance.await()；  // await应该放在上面形式中的循环判断中
        }
        // transfer
        // 其他操作
        conditionInstance.signalAll();  // 通知这个条件等待集的线程并激活
    }
}
```

signalAll方法激活等待这个条件的所有进程，当这些等待集中的线程从等待集中移除时，再次成为可运行的线程，调度器再次将他们激活。同时，这类线程会重新尝试进入该对象，一旦锁可用，便可以从await调用返回得到这个锁并从之前暂停的地方继续运行。由于调用了await挂起的线程没办法再自行激活，因此等待的线程寄希望于某个线程调用signalAll方法来唤醒他们，这及其重要。如果没有其他线程来激活等待的线程，那么将会出现死锁。如果所有其他线程都进入了await挂起阻塞，且最后一个线程挂起的时候并未发送signalAll方法解除其他线程挂起阻塞，这个线程也阻塞了，再没有线程可以解开挂起阻塞状态，程序会永远挂起。

**调用signalAll的时机**：只要一个对象的状态有变化，且有利于等待的线程，就可以使用signalAll方法。

signalAll方法不会立即激活一个等待的线程，只是解除等待线程的阻塞，使这些线程可以在当前线程释放锁之后，竞争访问对象

另外还有一个signal方法，随机选择等待集中的一个线程，并解除这个线程的阻塞状态，这样会比signalAll更高效。但是如果选择的线程仍然不能运行，继续进入阻塞。如果没有线程再调用signal方法的话，会进入死锁的情况。

> 只有当线程拥有一个有条件的锁时，才可以调用await、signalAll和signal方法

**虽然条件对象和锁机制的加入，是得资源访问更加安全，但同时还是牺牲了一定的效率。同时，条件对象的条件选择也很有挑战性**

**类`java.util.concurrent.locks.Lock`**:

- `Condition newCondition()`返回一个与这个锁相关联的条件对象

类`java.util.concurrent.Condition`:

- `void await()`将线程放在这个条件的等待集中
- `void signalAll()`将这个条件集中的所有等待集阻塞状态解除
- `void signal()`随机选取这个条件等待集的一个线程解除阻塞

##### **锁和条件对象的总结**

- 锁是用来保护代码片段的,一次只有一个线程可以执行被保护的代码片段
- 当锁被某个线程获取时,其他的线程都处于阻塞状态,只能等候锁被释放才能继续抢夺锁资源从而运行
- 锁可以管理试图进入被保护代码片段的线程
- 每个锁对象可以有多个条件对象
- 条件对象的await方法是用来挂起不符合条件的线程,挂起的线程被放在条件所属的等待集中
- 条件对象的signalAll和signal方法用来解除等待集中的线程
- 如果所有线程都处于等待集中,就会出现死锁的情况
- 条件对象是用来管理那些进入保护代码块但还不能运行的线程

#### synchronized关键字

Lock和Condition接口允许程序员充分控制锁定代码块.不过大多数情况下,并不需要那样充分的控制.完全在这个时候可使用Java内置的一种机制.1.0开始,Java中的每个对象都有一个内部锁.如果一个方法声明为synchronized,那么锁将保护整个方法.也就是如果线程想要访问这个方法,首先必须获得内部对象锁也就是下面的代码时等同的:

```java
public synchronized void method(){
    method body;
}
// 实际上就是内部类自定义了一个内在的锁,称之为intrinsicLock
// 那么按照加锁和条件对象的情况,上面方法可以修改为
public void method(){
    intrinsicLock.lock();
    try{
        while (!condition){
            intrinsicCondition.await();
        }
        method body
        intrinsicCondition.signalAll();
    }finally{
        intrinsicLock.unlock();
    }
}
```

实际中,对于条件对象的处理,使用了synchronized关键字的方法.条件对象和synchronized关键字的对应关系:

- `Condition.await()`-->`wait()`
- `Condition.signalAll()` --> `notifyAll()`
- `Condition.signal()` --> `notify()`

```java
public void method(){
    try{
        while (!condition){
            wait();  // 阻塞线程进入该条件的等待集
        }
        method body;
        notifyAll(); // 通知所有等待集的线程解除阻塞
    }finally{
    }
}
```

声明了synchronized关键字的方法可以得到更加简洁的代码.	原因就是每个对象内部都存在一个内部锁和一个内部条件,这个锁会管理使用synchronized的方法的线程,这个条件会管理调用了wait方法的线程

对于静态方法也是可以声明为同步的。如果调用声明为静态同步方法，此方法所处类的Class对象的锁会锁定，因此，没有其他线程可以调用这个类的该方法或者其他任何同步静态方法

内部锁和条件存在一定的局限性（synchronized）：

- 不能中断一个正在尝试获得锁的线程
- 不能指定尝试获得锁时的超时时间
- 每个锁仅有一个条件可能不够

对于Lock/condition和synchronized的选择：

- 最好两个都不使用，许多情况下，可以使用`java.util.concurrent`包中的一些机制，他会为你处理所有的锁定
- 如果synchronized关键字适合你的程序，就使用这种写法，应为可以减少代码数量，还可以减少出错的概率
- 如果特别需要Lock/Condition结构提供的一些额外能力，就可以使用这个

关于线程安全的Object类中的一些相关方法：

- `void notifyAll()`解除在这个对象调用wait方法的那些线程的阻塞状态,只能在同步方法或者同步代码块中使用,如果当前线程不是对象锁的拥有者,该方法抛出出`IllegalMonitorStateException`异常
- `void notify()`随机选择这个对象上调用了wait方法的线程,解除阻塞状态.只能在同步方法或者同步代码块中使用,如果当前线程不是对象锁的拥有者,该方法抛出`IllegalMonitorStateException`异常
- `void wait()`导致一个线程进入阻塞等待状态,直到得到通知,只能在同步方法或者同步代码块中使用,如果当前线程不是对象锁的拥有者,该方法抛出出`IllegalMonitorStateException`异常
- `void wait(long millis)`导致一个线程进入阻塞等待状态,直到得到通知或者经过了指定时间,只能在同步方法或者同步代码块中使用,如果当前线程不是对象锁的拥有者,该方法抛出出`IllegalMonitorStateException`异常
- `void wait(long millis, int nanos)`导致一个线程进入阻塞等待状态,直到得到通知或者经过了指定时间,只能在同步方法或者同步代码块中使用,如果当前线程不是对象锁的拥有者,该方法抛出`IllegalMonitorStateException`异常.纳秒数不能超过1000000

#### 同步块

还有一个机制可以获得锁,就是进入一个同步块:

```java
synchronized (onj)  // 这是进入同步块的语句语法,这个语法会获得obj的锁
{
    critical section;
}
```

#### volatile关键字

当只有几个变量需要保证同步安全时，此时若是再去使用锁来保证会导致资源浪费，开销不划算，因此，可以采用volatile关键字来保证。

volatile关键字为示例字段的同步访问提供了一种免锁机制。当声明一个字段为volatile时，虚拟机和编译器就知道这个字段可能被其他线程访问更新。

> 同步格言：当写一个变量接下来会被其他线程访问或者在你读取某一个变量而这个变量已经被其他线程已经改写时，则必须使用同步来保证变量的安全性。

#### final变量

可以使用final变量来保证安全的访问一个共享变量`final var accounts = new HashMap<String, Double>()`

使用这个方法可以保证其他线程在构造器完成构造之后才看到这个值，如果不使用final，不能保证其他线程看到的是accounts更新后的值，可能只是还未构造好的值null。

但是只能保证安全访问，但是并不能保证安全的操作该变量，可能还需要使用同步（线程安全的集合）才能保证安全

#### JUC包

原子操作：不会受到线程调度器的影响，只要开始了该操作就会一直运行到结束，中间不会有任何的线程切换。

原子性：在`java.util.concurrent.atomic`包中有许多的类提供了高效的机器级指令来保证操作的原子性

JUC包里面还提供了许多的Java并发操作的类和包：执行器类、Callable、Runnable以及保证线程安全的集合等等。

#### 死锁

尽管线程同步策略已经可以保证线程的资源安全，但是，可能会出现死锁的情况。

死锁：在并发计算中，死锁是线程或进程之间因相互等待而造成的全部阻塞，导致程序挂起，从而导致死锁。阻塞原因可能是等待线程发出信号或者释放锁。操作系统中，当一个进程或者线程由于请求的资源被另外一个等待的进程或者线程持有时，这个进程或者线程也进入等待状态，若此时另外一个线程或者进程也处于此种等待状态，将会导致整个系统中所有线程或进程全部进入等待阻塞状态，从而导致程序的挂起，引起死锁。

死锁在Java中并没有什么很好的解决方案和应对措施，只能依靠编程时设计考虑周到避免死锁发生。

#### 监视器

监视器概念最早1970年代提出来的。Java术语来讲，监视器有以下的几个特性：

- 监视器是只包含私有字段的类
- 监视器类的每个对象有一个关联的锁
- 所有的方法由这个锁来锁定
- 锁可以有任意多个相关联的条件（早期版本只有一个条件）

Java中的synchronized关键字就是实现了监视器的概念，但是相比较监视器概念来看，Java版监视器有几个差异：

- Java不要求字段是私有private
- 只有一个相关联的条件
- 方法不要求是synchronized
- 内部锁对客户是可用（Java可以使用其他对象的锁来实现线程安全->客户端锁定）

#### 禁用stop、suspend和resume的原因

对于stop来说，天生不安全，因为停止了线程之后，可能造成对象的损坏，这极大的影响了安全性

suspend：如果使用suspend挂起持有锁的线程，可能造成程序恢复之前锁不可用。如果使用suspend线程试图获取同一个锁，程序死锁：挂起的线程等待恢复，将其挂起的线程等待锁

#### 线程局部变量

线程间共享变量的风险：变量的不安全，资源的争夺造成的死锁

为了避免共享变量，可能需要使用线程局部变量ThreadLocal辅助类未各个线程提供各自的实例。

`java.lang.ThreadLocal<T>`类

- `T get()`得到这个线程的当前值，如果首次调用，会使用initialize来得到这个值
- `void set(T t)`为这个线程生成一个新值
- `void remove()`删除对应线程的值
- `static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier)`创建一个线程局部变量，其初始值通过调用给定的提供者生成

`java.util.concurrent.ThreadLocalRandom`

- `static ThreadLocalRandom current()`返回特定于当前线程的Random类的实例