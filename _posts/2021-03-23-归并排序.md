---
title: 归并排序算法
author: lijiabao
date: 2021-03-23 22:30
categories: ["DataStruture and Alogrithm"]
tags: ["DataStruture and Alogrithm"]

---

介绍一下归并排序，归并排序就是利用两个有序的数组归并成一个更大的有序数组的操作发明的一种简单递归排序算法

## 归并排序

归并排序能够保证任意长度为N的数组排序所需时间和NlogN成正比

缺点是它所需要的额外空间和N成正比。

实现归并的一种直截了当的方法是将两个不同的有序数组归并到第三个数组红，两个数组中的元素应该都实现Comparable接口。实现方法就是创建一个合适大小的数组然后将两个输入数组的元素一个个从小到大的放入数组中。

### 原地归并抽象方法

当对大数组进行排序需要多次归并，因此每次归并都创建一个新数组来存储结果会带来内存空间问题

希望有一种方法可以原地归并，可以将前半部分排序，再将后半部分排序，在数组中移动元素不需要使用额外空间。

尽管实际的实现非常复杂，特别是和使用额外空间的方法相比，但是，将原地归并抽象化仍然有帮助。方法签名merge(a, lo, mid, hi)代码实现如下：

下面代码使用了四个条件判断：左半边取完（开始取右半边元素），右半边取完（开始取左半边），右半边数组元素小于左半边（取右半边插入）右半边大于左半边元素（取左半边元素）

```java
public static void merge(Comparable[] a, int lo, int mid, int hi){
    int i = lo, j = mid + 1;
    Comparable[] au = new Comparable[hi];
    for (int k = lo;K<=hi;K++){
        au[k] = a[k];
    }
    for (int k = lo;K<=hi;k++){
        // 当第一个数组元素添加完毕之后，开始插入第二个数组元素
        if (i > mid) a[k] = au[j++];
        // 第二个数组元素插入完毕，插入第一个数组内的元素
        else if (j > hi) a[k] = au[i++];
        // 判断是否第二个数组的元素小于第一个数组
        else if (less(au[j], au[i])) a[k] = au[j++];
        // 不满足以上任何条件，从第一个数组选择元素插入
        else a[k] = au[i++];
    }
}
```

### 自顶向下的归并排序

基于原地归并的抽象方法实现另外一种递归归并，这是高效算法设计中分治思想的典型例子

```java
public class MergeSorted{
    private static int[] au;
    // 原地归并的方法实现
    public static void merge(int[] a, int lo, int mid, int hi){
    int i = lo, j = mid + 1;
    int[] au = new int[hi];
    for (int k = lo;K<=hi;K++){
        au[k] = a[k];
    }
    for (int k = lo;K<=hi;k++){
        // 当第一个数组元素添加完毕之后，开始插入第二个数组元素
        if (i > mid) a[k] = au[j++];
        // 第二个数组元素插入完毕，插入第一个数组内的元素
        else if (j > hi) a[k] = au[i++];
        // 判断是否第二个数组的元素小于第一个数组
        else if (less(au[j], au[i])) a[k] = au[j++];
        // 不满足以上任何条件，从第一个数组选择元素插入
        else a[k] = au[i++];
    }
    }
    // 排序
    public static void sort(int[] a){
        // 初始化辅助数组
        au = new int[a.length];
        // 调用递归归并排序方法对数组进行排序
        sort(a,0,a.length-1);
    }
    // 递归归并排序
    public static voidsort(int[] a,int lo, int hi){
        if (lo <= hi) return;
        int mid = lo + (hi - lo) / 2;  // 使用中间节点递归排序
        sort(a,lo,mid);  // 左边排序
        sort(a,mid,hi);  // 右边排序
        merge(a,lo,mid,hi);  // 归并排序
    }
}
```

### 自底向上的归并排序

递归实现的归并排序时算法设计分治思想典型应用，将一个大问题分割成小问题分别解决，然后用小问题来解决大问题。

但是往往归并的数组大多数都非常小。因此就产生了另外一种方法：先归并那些简单的微型数组，然后再成对归并得到的子数组，知道我们将整个数组归并到一起。

这种新的实现方法比标准的递归方法代码量更少，就是1、2、4、8、16依次提升规模：

```java
public class MergeBU{
    private static int[] waitSortArray;
    private static int[] au;
    static {
        waitSortArray = new int[]{1, 3, 2, 6, 9, 8, 5, 4, 7};
    }
    // 原地归并的方法实现
    public static void merge(int[] a, int lo, int mid, int hi){
        int i = lo, j = mid + 1;
        int[] au = new int[hi];
        for (int c=lo;c<=hi;c++){
            au[c] = a[c];
        }
        for (int c=lo;c<=hi;c++){
            // 当第一个数组元素添加完毕之后，开始插入第二个数组元素
            if (i > mid) a[c] = au[j++];
                // 第二个数组元素插入完毕，插入第一个数组内的元素
            else if (j > hi) a[c] = au[i++];
                // 判断是否第二个数组的元素小于第一个数组
            else if (less(au[j], au[i])) a[c] = au[j++];
                // 不满足以上任何条件，从第一个数组选择元素插入
            else a[c] = au[i++];
        }
    }
    // 排序
    public static void sort(int[] a){
        int N = a.length;
        // 初始化辅助数组
        au = new int[N];
        // 使用自底向上递归归并排序
        for (int sz=1;sz<N;sz+=sz){
            for (int lo=0;lo<N-sz;lo += sz+sz){
                merge(a, lo, lo+sz-1,Math.min(lo+sz+sz-1,N-1))
            }
        }
    }

    public static void main(String[] args) {
        for (int i: waitSortArray) {
            System.out.print(i);
        }
        sort(waitSortArray);
        System.out.println();
        for (int i : waitSortArray){
            System.out.print(i);
        }
    }
}
```

自底向上的归并排序适合用链表组织的数据。这种方法只需要重新组织链表链接就能将链表原地排序，不需要创建任何新的链表结点。

### 归并排序的优化



#### 对小规模数组使用插入排序

使用不同方法处理小规模问题可以改进大多数递归算法的性能，因为递归会使小规模问题中方法的调用过于频繁，所以改进对他们的处理方法就能改进整个算法

比如插入排序在小数组规模问题上（确定使用插入排序的一个问题规模，如15长度的数组）很可能比归并算法更快，适当的使用插入排序会使整体算法性能更好

#### 对数组有序性进行判断

可以添加判断条件（数组有序，a[i] < a[i+1]），这时认为数组已经有序，就可以跳过某些方法的处理。改动并不影响排序的递归调用，但是任意有序子数组的算法运行时间就变为线性的了

#### 不将元素复制到辅助数组

可以节省将数组元素复制到用于归并的辅助数组的时间（空间必须要）

要做到上述操作，我们可以采用两种排序方法：

- 将数组从输入数组排序到辅助数组
- 将数据从辅助数组排序到输入数组（需要一些技巧，在递归调用的每个层次交换输入数组和辅助数组的角色）

> 研究新问题时，最好的方法时先实现一个你能想到的最简单的程序，当它成为瓶颈时再去改进。
>
> 实现那些只能把运行时间缩短某个常数因子的改进措施并不值得



## 排序算法的复杂度

>  **没有任何基于比较的算法能够保证使用少于lg(N!) ~NlgN次比较将长度为N的数组排序。**

这个结论告诉我们在设计排序算法时能够达到的最佳效果。如果没有这个结论，我们可能尝试设计一个最坏情况下比较次数只有归并排序的一般的基于比较的算法。但是结论告诉我们这样的算法并不存在。这是一个重要结论，适用于任何基于比较的算法。

**归并排序的最坏情况下的比较次数为NlgN。**

> 归并排序时一种渐进最有的基于比较排序的算法。归并排序在最坏情况下的比较次数和任意基于比较的排序算法所需最少比较次数都是NlgN。

归并排序的最优性并不是结束。这算法的理论还是有**不少局限性：**

- 归并排序的空间复杂度不是最优的
- 实践中不一定遇到最坏情况
- 除了比较操作，算法的其他操作（如访问数组）也可能很重要
- 不进行比较也能将某些数据排序