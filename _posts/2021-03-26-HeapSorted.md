---
title: 堆排序
author: lijiabao
date: 2021-03-26 14:30
categories: ["排序", "DataStruture and Alogrithm"]
tags: ["排序", "DataStruture and Alogrithm"]
---

介绍一下基于堆实现的优先级序列如何应用在排序中来，以及堆排序的一些优点和缺点

## 堆排序

对于优先级队列的每一次种实现，都可以将其变为一种排序方法。将所有元素放入到一个查找最小或者最大元素的优先级队列，然后重复调用删除最小元素或者删除最大元素的操作按照顺序删除，就得到了排序好的元素。

使用无序数组实现的优先队列这么实现相当于进行了一次插入排序。而基于堆实现的优先级队列的该操作就实现了堆排序。

堆排序可以分为两个阶段：

- 堆构造阶段：将原始数据重新组织安排到一个堆中
- 下沉排序阶段：从堆中按照递减顺序取出所有元素并得到排序结果

### 堆的构造

将N个给定元素构造一个堆：很容易在NlogN成正比的时间内完成这项任务，只需要从左至右遍历数组，然后使用上浮操作保证指针扫描左侧的所有元素已经是一棵堆有序的完全树即可，就和连续向优先队列插入元素一样

另外一种比较快速聪明的方法：从右至左使用下沉函数构造子堆。

```java
public class HeapSorted<Key extends Comparable<Key>>{
    private Key[] pq;  // 创建一个数组用来创建基于堆实现的完全二叉树
    private int N = 0;   // 队列大小，只是用[1..N]，0不使用
    public MaxPQ(int max){
        pq = (Key[]) new Comparable[max+1];
    }
    // 堆排序算法的实现
    public static void sort(Comparable[] a){
        int N = a.length;  // 获取数组大小，然后利用该数值来创建队列
        for (int k=N/2;k>=1;k--)
            sink(a,k,N);  // 对数组进行树的上下遍历然后进行下沉恢复有序性
        while (N > 1){
            // 将最大元素和最后元素交换位置，并进行下沉恢复有序性
            exch(a, 1, N--);
            sink(a, 1, N);
        }
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
    public void sink(Comparable[] a, int k, int aLen){
        // 判断是否父结点小于子结点或者当前结点没有子结点了
        while (2*k<=aLen){
            int j = 2*k;
            if (j<N && less(j, j+1)) j++;
            if (!less(k,j)) break;
            exch(a,k,j);
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
    public void exch(Comparablr[] a, int i, int j){
        Key temp = a[i];
        a[i] = a[j];
        a[j] = temp;
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

**上述的排序代码实现：**首先开始从二叉树的底部之上的最小子结点开始，进行下沉操作，从下部开始往上部进行二叉堆的有序性的恢复操作。一直进行下沉操作直到到达根节点完成对的有序化操作。待到堆有序之后，开始交换最大值和堆数组的最后一个元素，将最大值放在数组的最右边。如此往复进行直到到达数组的1号元素结束操作，此时的数组已经按照从小到大的顺序排列好了。

### 下沉排序

堆排序的算法中排序的操作主要是在第二阶段完成的，也就是第二部分代码交换最大元素和尾元素。

将堆中的最大元素删除，然后放入堆缩小后数组空出的位置。这个过程和选择排序类似，但需要的比较少很多，堆排序提供了从未排序部分找到最大元素的有效方法。

> 将N个元素排序，堆排序只需要少于（2NlgN+2N)次比较（一半次数的交换）
>
> 2N项来自堆的构造。2NlgN在于每次下沉操作可能需要2lgN次比较

### 堆排序的改进（先下沉后上浮）

大多数下沉排序期间重新插入堆的元素会被直接加入到堆底。Floyd发现，可以通过免去检查元素是否到达正确位置来节省时间。**在下沉中总是直接提升较大的子结点直至到达堆底，然后使元素上浮到正确位置**。<font color='red'>这可以减少一半的比较次数，接近归并排序所需要的比较次数。但是这种方法需要额外的空间，只有实际应用中比较操作代价较高时才有用。</font>

堆排序在排序复杂性的研究中有重要的地位，因为它是唯一能够同时最优利用空间和时间的方法，最坏情况下，也能保证使用2NlgN次比较和恒定的额外空间。<font color='red'>空间比较紧张的时候很流行（嵌入式系统或低成本的移动设备），因为只用几行就实现较好的性能。</font>

但是，<font color='red'>因为他无法使用缓存，现代系统许多应用很少使用。数组元素很少和相邻元素进行比较，因此缓存未命中次数远高于大多数都在相邻元素进行比较的算法：快速排序、归并排序甚至希尔排序</font>

<font color='red'>堆实现的优先级队列在现代应用程序越来越重要，因为能够在插入操作和删除最大元素的操作场景中保证对数级别的运行时间。</font>