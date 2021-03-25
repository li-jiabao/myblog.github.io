---
title: 优先级队列和堆算法
author: lijiabao
date: 2021-03-25 22:30
categories: ["DataStruture and Alogrithm"]
tags: ["DataStruture and Alogrithm"]
---

介绍一下优先级队列的概念以及优先级队列的简单实现，接着介绍了堆的定义以及如何使用堆来实现优先级队列，其中还介绍了一些可能的堆算法的改进。

## 优先级队列

很多时候你并不需要全部有序、或者不一定要一次排序好。

类似像一个同时运行多个应用程序的电脑，这时候你需要未每个应用程序事件分配一个优先级，并总是处理优先级高的事件。

类似上面情况，需要一个合适的数据结构，应该支持两种操作：删除最大元素和插入元素。这一类数据结构类型叫做优先队列。优先队列使用类似队列和栈，不过想要高效实现优先级队列更具挑战性。

优先队列的一些重要的应用场景包括模拟系统，其中事件的键几位发生的时间，系统需要按照时间顺序处理所有时间；任务调度，其中的键值对应的优先级决定了应该先执行哪些任务。

### 优先级队列API和简单用例分析

优先级队列是一种抽象数据类型，表示了一组值和对这些值的操作，他的抽象层是我们能够方便将应用程序和各种具体的实现隔离开来。首先应该给数据结构的用例详细定义一组应用程序接口（API）。优先级队列最重要的操作就是删除最大元素和添加元素，因此会更关注这两个，但是还需要一些其他的API：比如辅助函数，测试函数等，这些就根据实际需要来进行API设计。

API设计的时候考虑到灵活性，可以设计为泛型。下面是优先级队列的API：

| API                                               | 描述                              |
| ------------------------------------------------- | --------------------------------- |
| `public class MaxPQ<Key extends Comparable<Key>>` | 创建一个优先级队列实现的类        |
| `MaxPQ()`                                         | 创建一个优先级队列                |
| `MaxPQ(int max)`                                  | 创建一个最大容量为max的优先级队列 |
| `MaxPQ(Key[] a)`                                  | 用a[]元素创建一个优先级队列       |
| `void insert(Key v)`                              | 向优先级队列插入一个元素          |
| `Key max()`                                       | 返回最大元素                      |
| `Key delMax()`                                    | 删除并返回最大元素                |
| `boolean isEmpty()`                               | 判断队列是否为空                  |
| `int size()`                                      | 返回优先队列元素个数              |

优先级队列实际使用中的算例代码：

```java
public class MinPQTest{
    public static void main(String[] args){
        int M = Integer.parseInt(args[0]);
        MinPQ<Transaction> pq = new MinPQ<Transaction>(M+1); // 创建最小优先级队列，指定容量
        while (StdIn.hasNextLine()){
            // 为下一行输入创建位置插入元素
            pq.insert(new Transaction(StdIn.readLine()));
            if (pq.size()>M){
                pq.delMin(); // 如果存在过多元素则删除最小元素
            }
        }// 读取内容结束后，留下的是最大的M个元素
        Stack<Transaction> stack = new Stack<Transaction>();
        while (!pq.isEmpty()) stack.push(pq.delMin());
        for (Transaction t:stack) System.out.println(t);
    }
}
```

找到N个输入中最大的M个元素的成本分析：

| 方法                 | 时间成本 | 空间成本 |
| -------------------- | -------- | -------- |
| 排序算法用例         | NlogN    | N        |
| 初级实现的优先队列   | NM       | M        |
| 基于堆实现的优先队列 | NlogM    | M        |

### 初级实现

可以使用有序数组、无序数组和链表来实现，队列较小时，大量使用两种主要操作之一或者操作元素的顺序已知，十分实用。

#### 数组实现（无序）

实现优先队列的最简单方法基于下压栈的实现，插入和栈的push不需要改变。需要实现删除最大元素，可以添加类似选择排序的内循环戴拿，将最大元素和边界元素交换，然后删除，和对栈的pop方法实现一样。同样的，也可以加入调整数组大小的代码来保证数据结构中至少含有四分之一的元素而永远不会溢出。

```java
/**
* 利用无序数组来实现优先级队列的操作。主要在于删除最大元素操作的实现
* 删除最大元素可以在方法体内部加上一层排序循环找到数组的最大元素并和数组某一侧元素交换位置
* 之后就可通过删除边界元素达成删除最大元素的操作
*/
public class NoSortedArrayPriorityQueue<Key extends Comparable<Key>>{
    private Key[] queue;
        public NoSortedArrayPriorityQueue(){
        // 实现无参数构造方法,默认容量10
        this.queue = (Key[]) new Comparable[10];
    }
    public NoSortedArrayPriorityQueue(int max){
        // 实现指定最大容量的构造方法
        this.queue = (Key[]) new Comparable[max];
    }
    public NoSortedArrayPriorityQueue(Key[] a){
        // 根据给定的数组元素创建优先级队列的构造方法
        this.queue = a;
    }
    // 实现插入方法
    public static void insert(int pos,Key ele){
        this.queue[pos] = ele;
    }
    // 实现删除最大元素的操作
    public Key pop(){
        int max = 0
        for (int i=0;i<queue.length;i++){
            if (queue[i].compareTo(queue[max])) max=i;
        }
        return del(max);
    }
    // 实现删除指定位置元素并返回的操作
    public Key del(int pos){
        a = this.queue[pos];
        this.queue[pos] = 0;
        return a;
    }
}
```



#### 数组实现（有序）

另外一种方法就是在插入的时候就把元素排序好，这样每次插入之后保证最大的元素在数组的一侧，这样优先级队列的删除最大元素的操作就和栈的删除操作pop一样了。

```java
/**
* 利用无序数组来实现优先级队列的操作。主要在于删除最大元素操作的实现
* 删除最大元素可以在方法体内部加上一层排序循环找到数组的最大元素并和数组某一侧元素交换位置
* 之后就可通过删除边界元素达成删除最大元素的操作
*/
public class SortedArrayPriorityQueue<Key extends Comparable<Key>>{
    private Key[] queue;
        public NoSortedArrayPriorityQueue(){
        // 实现无参数构造方法,默认容量10
        this.queue = (Key[]) new Comparable[10];
    }
    public NoSortedArrayPriorityQueue(int max){
        // 实现指定最大容量的构造方法
        this.queue = (Key[]) new Comparable[max];
    }
    public NoSortedArrayPriorityQueue(Key[] a){
        // 根据给定的数组元素创建优先级队列的构造方法
        this.queue = a;
    }
    // 实现插入方法
    public void insert(int pos,Key ele){
        // 利用插入排序算法逻辑将插入元素按从大到小排序
    }
    // 实现删除最大元素的操作（因为插入就已经拍好顺序，只要删除第一个即为最大元素）
    public Key pop(){
        return this.queue[0]
    }
}
```

#### 链表实现

可以基于链表来实现，只需要修改pop方法找到并返回最大元素，或者也可以在push方法中保证元素按照顺序排列，然后再使用正常的pop操作删除首元素（也就是最大元素）

```java
public class LinkedPriorityqueue<Key extends Comparable<Key>>{
    private static Node first;
    
    // 定义结点类，用来
    private class Node{
        Key item;
        Node next;
    }
    
    // 添加元素
    public void push(Key item){
        Node oldFirst = first;
        first.item = item;
        first.next = oldFirst;
        // if (isEmpty()){}
    }
    // 删除最大元素
    public Key pop(){
        Node max=first;
        for (Node node=first;node!=null;node=first.next){
            if (node.item.compareTo(max.item)>0) max=node;
        }
        return del(max);
    }
    // 实现删除任意链表元素额方法
    public Key del(Node node){
        // 删除任意位置链表元素的实现
    }
}
```

**优先级队列再最坏情况下的时间复杂度**

| 数据结构 | 插入元素 | 删除最大元素 |
| -------- | -------- | ------------ |
| 有序数组 | N        | 1            |
| 无序数组 | 1        | N            |
| 堆       | logN     | logN         |
| 理想情况 | 1        | 1            |

## 堆

数据结构的二叉堆能够很好的实现优先级队列的基本操作。二叉堆的数组中，每个元素都要保证大于等于另外两个特定位置的元素，这些位置的元素又要至少大于等于数组中另外两个元素。以此类推。可以将所有元素画成一棵二叉树，将每个较大元素和两个较小元素用边链接就可以很容易看出这种结构

> 当二叉树的每个结点都大于等于她的两个子结点时，称为堆有序

根据堆有序的性质可以知道：堆有序的二叉树中，每个结点都小于等于它的父结点。其中二叉树的根节点元素最大。

### 二叉堆表示法

如果用指针表示堆有序的二叉树，那么每个元素都需要三个指针来找到它的上下结点（父节点和两个子结点各一个）

如果使用完全二叉树，表达会变得特别方便，如下图：要画出这样一棵完全二叉树，可以先定下根结点，然后一层层的从上往下，从左往右，在每个结点下方链接两个更小子结点，直至所有结点全部连接完毕。

![](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E5%A0%86%E6%9C%89%E5%BA%8F%E7%9A%84%E5%AE%8C%E5%85%A8%E4%BA%8C%E5%8F%89%E6%A0%91.png)

完全二叉树可以只使用数组而不需要使用指针就可以表示。方法：将二叉树的结点按照层级顺序放入数组中，根节点在位置1，两个子结点位置2和3，而子结点的位置分别在4、5和6、7，以此类推，直到把所有元素排好。

> 二叉堆是一组能够用堆有序的完全二叉树排序的元素，并在数组中按照层级储存（不使用数组的第一个位置）

在一个堆中，位置k的结点的父结点的位置为[k/2]，子结点的位置则分别为2k和2K+1。这样可以在不使用指针的情况下，可以通过计算数组的索引在数的上下移动：k上一层则为[k/2]下一层为2k或者2k+1。

数组实现的完全二叉树结构很严格的，但是灵活性已经让我们可以高效优化优先级队列。使用二叉堆可以实现对数级别的插入和删除最大元素的操作。

> 一棵大小为N的完全二叉树的高度为lgN。N达到2的幂时数的高度会加一。

### 堆算法

一个大小为N的堆实现需要使用N+1长度的数组，因为数组的第一个位置不使用。

堆的有序化：就是指将一个打乱顺序的二叉堆的状态恢复的这个过程

堆有序化过程会遇到两种情况：

- 某个结点优先级上升（或者堆底加入一个新元素时）：需要从下往上恢复顺序
- 某个结点优先级下降（根节点替换为较小元素时）：需要从上往下恢复顺序

#### 由下往上的堆有序化（上浮）

如果堆的状态因为某个结点变得比它的父结点更大而打破，那就需要将该结点和父结点交换位置，当然这个时候还会出现这个结点仍然比调换之后的父结点更大，不断操作直到小于当前父结点或者没有父结点成为根节点。实现代码：

```java
// 当子结点大于父结点时，进行上浮操作
public void swim(int k){
    // 判断是否到了根节点或者字节结点小于父结点
    while (K>1 && less(k/2, k)){
        exch(k/2,k);
        k /= 2;
    }
}
// 交换堆中的两个元素
public void exch(int i, int j){
    Key temp = pq[i];
    pq[i] = pq[j];
    pq[j] = temp;
}
// 比较两个元素
public boolean less(int i, int j){
    return pq[i].compareTo(pq[j]) < 0;
}
```

#### 由上往下的堆有序化（下沉）

当堆的有序性因为某个结点变得比它的子结点更小而被打破时，这时候可通过将该节点和它的子结点调换来重新恢复，如果调换之后仍然小于子结点，继续进行该操作调换即可，操作一直进行到没有子结点或者大于子结点。实现代码：

```java
// 当父结点小于字结点的时候进行下沉操作
public void sink(int k){
    // 判断是否父结点小于子结点或者当前结点没有子结点了
    int N = pq.length;
    while (2k<=N){
        int j = 2*k;
        if (j<N && less(j, j+1)) j++;
        if (!less(k,j)) break;
        exch(k,j);
        k = j;
    }
}
```

#### 删除增加元素的实现

增加和删除元素的实现将会依靠上述的两种恢复有序化的手段：

- **插入元素**：将元素加到数组末尾，增加堆的大小并让这个元素上浮到合适的位置

- **删除最大元素**：从数组的顶端删除最大元素并将数组的最后一个元素放到顶端，较少堆的大小并让这个元素下沉到合适位置。

下面是基于堆的优先级队列的代码实现以及一个简单的测试：

```java
public class MaxPQ<Key extends Comparable<Key>>{
    private Key[] pq;  // 创建一个数组用来创建基于堆实现的完全二叉树
    private int N = 0;   // 队列大小，只是用[1..N]，0不使用
    public MaxPQ(int max){
        pq = (Key[]) new Comparable[max+1];
    }
    public Key delMax(){
        // 找到队列中最大元素删除返回将最后一个元素放到顶端
        Key max = this.pq[1];
        exch(1,N--);
        pq[N+1] = null;
        sink(1);
        return max;
    }
    public void insert(Key a){
        // 向队列中插入元素
        pq[++N] = a;
        swim(N);
    }
    public boolean isEmpty(){
        //判断队列是否为空
        return N == 0;
    }
    public Key max(){
        // 返回队列最大的元素
        return this.pq[1];
    }
    public int size(){
        // 返回队列的元素个数
        return N;
    }
    // 下沉操作
    // 当父结点小于字结点的时候进行下沉操作
    public void sink(int k){
        // 判断是否父结点小于子结点或者当前结点没有子结点了
        while (2*k<=N){
            int j = 2*k;
            if (j<N && less(j, j+1)) j++;
            if (!less(k,j)) break;
            exch(k,j);
            k = j;
        }
    }
    // 当子结点大于父结点时，进行上浮操作
    public void swim(int k){
        // 判断是否到了根节点或者字节结点小于父结点
        while ( k>1 && less(k/2, k)){
            exch(k/2,k);
            k /= 2;
        }
    }
    // 交换堆中的两个元素
    public void exch(int i, int j){
        Key temp = pq[i];
        pq[i] = pq[j];
        pq[j] = temp;
    }
    // 比较两个元素
    public boolean less(int i, int j){
        return pq[i].compareTo(pq[j]) < 0;
    }
    // 测试代码
    public static void main(String[] args) {
        MaxPQ<Integer> queue = new MaxPQ(10);
        for (int i=1;i<11;i++){
            Integer ele = (int)(10*Math.random());
            System.out.print(ele + " ");
            queue.insert(ele);
        }
        System.out.println();
        for (int i=0;i<10;i++){
            System.out.print(queue.delMax()+" ");
        }
    }
}
```

> 对于一个含有N个元素的基于堆的优先队列，插入元素操作只需要不超过(LgN+1)次比较，删除最大元素的操作需要不超过2lgN次比较
>
> 证明：两种操作都是在根节点和堆底之间移动元素，路径的长度不超过lgN。对于路径上的结点，删除最大元素需要两次比较，一次用来找出较大的子结点，一次用来确定该子结点是否需要上浮

**这就意味着性能的突破，因为基于数组实现的优先队列的初级实现总是需要线性时间来完成其中一种操作，但是基于二叉堆的实现能够保证在对数时间内完成它们，这种差别就可以使得我们解决之前不能解决的问题。**

### 堆算法的一些改进操作

#### 多叉堆

基于数组表示的完全三叉树构造退并修改相应代码是可以实现的。对于数组N个元素，位置k的结点大于等于位于（3k-1，3k，3k+1）的结点，小于等于位于（k+1）/3的结点。

当然对于给定的d，将其实现为任意的d叉树也是可以的，只是需要在树高logdN和每个结点的d个字节带你找到最大者的代价之间找到折中，这取决于实现的细节以及不同操作的预期相对频繁程度

#### 调整数组的大小

可以添加一个没有参数的构造函数，在insert方法中添加将数组长度加倍的方法，在delMax方法中添加将数组长度减半的代码。

如果实现了这样的操作，算法的用例就无需关注各种队列大小的限制。当优先级队列数组大小可以调整、队列长度可以是任意值时，上面提到的命题指出的对数时间复杂度就只是针对一般性的队列长度N来说的了。

**数组大小调整的实现：**当队列满了，就将队列扩容至两倍，如果队列长度小于总长度的1/4，就缩减队列大小为1半，这样既可以释放空间，同时还能保证缩减之后还可以进行多次添加删除操作。

#### 元素的不可变性

优先级队列存储的是用例创建的对象，假设用例代码不会修改他们（修改可能会造成堆的有序性受到破坏），为了保证更好的堆的稳定性，可以把不可修改变成强制条件。但是往往并不会这样做，因为这样做会增加代码复杂性从而影响程序性能。

#### 索引优先队列

为了可以引用已经进入优先队列中的元素是有必要的，做到这一点可以给每个元素一个索引。

下面是泛型索引优先级队列的API：

| API                             | 描述                                                        |
| ------------------------------- | ----------------------------------------------------------- |
| `public IndexMinPQ(int max)`    | 创建一个最大容量为maxN的优先队列，索引的取值范围为0到maxN-1 |
| `void insert(int k, Item item)` | 插入一个元素，将它和索引k相关联                             |
| `void change(int k, Item item)` | 将索引为k的元素设置为item                                   |
| `boolean contains(int k)`       | 是否存在索引为k的元素                                       |
| `void delete(int k)`            | 删除索引k及其相关联的元素                                   |
| `Item min()`                    | 返回最小元素                                                |
| `int minIndex()`                | 返回最小元素的索引                                          |
| `int delMin()`                  | 删除最小元素并返回它的索引                                  |
| `boolean isEmpty()`             | 优先队列是否为空                                            |
| `int size()`                    | 优先队列的元素数量                                          |

> 在一个大小为N的索引优先队列中，插入元素、改变优先级、删除和删除最小元素操作所需的比较次数和logN成正比
>
> 证明：因为堆中所有路径最长为lgN

索引优先级队列的所有操作在最坏情况下的成本：

| 操作         | 比较次数的增长数量级 |
| ------------ | -------------------- |
| `insert()`   | logN                 |
| `change()`   | logN                 |
| `contains()` | 1                    |
| `delete()`   | logN                 |
| `min()`      | 1                    |
| `minIndex()` | 1                    |
| `delMin()`   | logN                 |

#### 索引优先级队列的多向归并用例

下面的代码用例使用索引优先级队列解决了多向归并问题：将多个有序的输入流归并成一个有序的输入流

```java
public class Multiway{
    public static void merge(In[] streams){
        int N = streams.length; 
        IndexMinPQ<String> pq = new IndexMinPQ<String>(N); 
        for (int i = O; i < N; i++) {
            if (!streams[i].isEmpty()) pq.insert(i, streams[i].readString()); 
        }
        while (!pq.isEmpty()) {
            StdOut.println(pq.min()); inti= pq.delMin(); 
            if (! streams[i].isEmpty()) pq.insert(i, streams [i]. readString());
        }
    }
    public static void main(String[] args){
        int N = args.length; 
        In[] streams= new In[N]; 
        for (int i = 0; i < N; i ++) 
            streams[i] = new In(args[i]);
        merge(streams); 
    }
}

// 运行命令：java Multiway ml.txt m2.txt m3.txt合并多个有序文件的输入流
```