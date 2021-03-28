---
title: 快速排序算法
author: lijiabao
date: 2021-03-24 21:30
categories: ["DataStruture and Alogrithm"]
tags: ["DataStruture and Alogrithm"]
mathjax: true


---

介绍一下快速排序算法，快速排序可能是应用最广泛的排序算法了。优点：实现简单，适用于各种不同的输入数据，一般应用中比其他算法都要快得多，且他是原地排序（只需要一个很小的辅助栈）将长度为N的数组排序所需时间和NlgN成正比，内循环比大多数排序算法都短小。

## 快速排序

尽管快速排序有比较多的优点，比如空间需求小，时间复杂度和NlgN成正比，内循环短小。但是快速排序算法缺点是很脆弱，实现中要非常小心才能避免低劣的性能。

### 基本算法

快速排序也是一种分治的算法。也是将一个大数组分成两个子数组，然后将两部分独立排序。

归并排序是将两个有序的子数组归并成一个有序数组，数组切分是等分为两半

快速排序是先将两个数组按照指定元素分为小于该元素的数组和大于该元素的数组，然后再去将子数组排序，排序好了之后，数组就是有序的了。快速排序的切分取决于数组的内容

切分算法的实现：一般策略，随意的选取a[lo]元素作为切分元素，这个元素也就是那个会排好的元素，然后从数组的左边开始寻找比该元素大的元素，而从数组右边找寻比该元素小的元素，然后将二者交换位置。这样一直遍历直到两个指针相遇，这时候将切分元素和左数组最右边的元素交换位置即可达成左数组全小于等于切分元素，右数组全大于等于切分元素，这时候，再将左右数组排序好之后，就完成了整个数组的排序。

```java
/**
*下面是切分算法的实现
*/
public static int partition(Comparable[] a, int lo, int hi){
    Comparable ele = a[lo];
    int i = lo,j=hi+1;
    // 算法的逻辑就是采用死循环，将不满足的异常条件设置成为跳出循环，然后其他条件就直接进行交换操作
    while (true){
        while (less(a[++i],ele)) if (i==hi) break;// i遍历到最高值结束
        while (less(ele,a[--j])) if (j==lo) break; // j遍历到最小值结束
        if (i>=j) break; // 遍历到i>=j结束
        exch(a,i,j); // 交换元素
    }
    exch(a,lo,j);  // 将切分元素放回正确位置
    return j;   // 返回切分元素的索引
}
// 排序方法
public static void sort(int[] array){
    sort(array, 0, array.length -1);
}
// 定义排序的方法体
public static void sort(int[] array, int lo, int hi){
    // if (hi <= lo+5) { InsertSorted.sort(array, lo, hi); return;}
    // 上面一句是将快排算法中遇见的小数组使用插入排序
    if (hi <= lo) return; // 正常的递归返回语句
    int j = partition(array, lo, hi);
    sort(array, lo, j-1);
    sort(array, j+1, hi);
}
```

### 注意事项

快速排序算法有一些地方很容易出现错误和降低性能的情况，下面将就这些问题进行简要的介绍，避免在快速排序中出现这些问题

#### 原地切分

如果使用辅助数组，可以很容易实现切分，但是切分后的数组复制回去的开销可能并不值得改进。

有时候如果将空数组创建再递归的切分方法中，会大大降低排序算法的性能

#### 越界

如果切分元素是数组中最小或者最大的那个元素，这时候就要小心别让扫描指针跑出数组的边界。因此在切分算法实现的时候可以使用明确的检测来预防这种情况。在算法优化的时候也可以找到那些冗余代码，这样也能增加算法的性能

#### 保持随机性

数组元素的顺序被随机打乱过的，这是为了保证算法计算时不会受到输入依赖的影响。这中保持随机性的操作对于预测算法的运行时间很重要。在partition切分算法中保持随机性的另外一种方法是随机的选择一个切分元素

#### 终止循环

**保证循环结束时需要格外小心**。同时正确的检测指针是否越界也需要一定的技巧，并不是看上去那么简单，最常见的错误是没有考虑到数组中可能包含和切分元素一样的值，这个问题在切分算法实现是也需要注意

#### 切分重复值情况

在实现判断的时候，最好时在遇到大于等于切分元素值的元素时停下，右侧扫描则是遇到小于等于切分元素时停下。尽管这样可能造成不必要的等值交换，但是在某些情况可以避免算法运行时间变成平方级别。

#### 终止递归

**保证递归的结束需要格外小心**。实现快速排序最常见的错误就是不能保证将切分元素放入正确的位置，从而导致程序在切分元素正好时子数组的最大或是最小值陷入无限递归循环中

### 性能特点

#### 内循环小

快速排序的切分方法的内循环用一个递增的索引将数组元素和一个定值比较，这个算法的内循环极小。像归并排序和希尔排序的一般比快速排序慢，因为在内循环中还会移动数据

#### 比较次数少

优点就是比较的次数相较于其他的排序算法来说算少的。快速排序算法的效率依赖切分数组的效果，这又取决于切分元素的值。

快速排序算法的最好情况是每次都正好将数组对半分，这种情况下，快速排序算法的比较次数正好满足分治递归的$C_N=2C_{N/2}+N$公式。$2C_{N/2}$表示将两个子数组排序的成本，N表示用切分元素和所有数组元素进行比较的成本。这个递归公式的解是$C^N$~$NlgN$。尽管不一定每次都落在中间，但平均而言切分元素都能落在数组的中间，如果将每个切分位置都考虑进来只会让递归更加复杂难解决，而且最终结果还是类似的。快速排序这么好用受推崇正是由于这个原因。

归并排序和快速排序都可以做到和NlgN成正比，但是快速排序一般更快，因为数据移动次数少。

#### 一些缺点

实际应用中可能会碰到元素重复的情况（精确分析较难），但可以证明即使存在重复的元素，平均比较次数也不会大于$C_N$

快速排序尽管存在很多优点，但是他还有潜在缺点：**切分不平衡是这个程序可能会十分低效**。当其中一个子数组较少另一个较多，这将会导致大数组需要切分很多次。因此快排排序前将数组随机排序主要就是为了避免这种情况，它可以是产生糟糕切分的可能性降到极低。

## 快速排序算法改进

快速排序的算法改进一直有人研究并改进，有许多是很不错的，但是有一些带来的性能提升可能被副作用所抵消。

如果排序代码被执行很多次或者被用在大型数组上（特别是会被发布为一个函数，排序的对象数组特性未知）下面的一些改进算法是值得参考的。

### 切换到插入排序

和大多数递归排序算法一样，改进排序算法性能的一个简单方法基于一下两点：

- 对于小数组，快速排序比插入排序慢
- 因为递归，快速排序的sort（）方法在小数组中也会调用自己

改进可以将算法中的递归判断语句`if (hi <= lo) return ;`改成`if (hi <= lo+M) { InsertSorted.sort(a, lo, hi); return;}`

### 三取样切分

使用子数组的一小部分元素的中位数来切分数组，这样做到的切分更好，但代价是需要计算中位数。**取样大小设置为3并使用大小居中的元素切分效果最好**。还可以将取样元素放在数组末尾作为哨兵来去掉partition()方法中的数组边界测试

### 熵最优的排序

实际应用可能会出现大量重复元素的数组，实现这样数组的快速排序算法还有很大的改进空间。例如，一个元素全部重复的子数组就需要继续排序了，但是标准的快速排序算法还会继续切分，这就浪费了性能。

当有大量重复元素情况，快排并不知道会继续切分，这就提供了改进的一个突破点。

一个简单的想法是将数组切分为三部分，分别对应小于等于和大于切分元素的子数组。这种切分比目前使用的二分法更加复杂。

其中一种解决是：从左到右遍历数组一次，维护一个指针lt使得a[lo..lt-1]中的元素都小于切分元素，一个指针gt使得a[gt..hi]中的元素都大于切分元素，一个指针i使得a[lt..i-1]中的元素都等于切分元素，a[i..gt]中的元素还未确定。

- a[i]小于v，将a[lt]和a[i]交换，将lt和i加一
- 大于v，将a[gt]和a[i]交换，将gt减一
- 等于v，将i加一

这些操作保证数组元素不变，且缩小gt-i的值（循环才会结束）。此外，除非和切分元素相等，其他元素都会被交换

<font color='red'>尽管这是一个解决方法，但是因为数组中重复元素不多的普通情况下，比标准的二分法使用了很多次交换。</font>

这种方法仅限于含有大量重复元素的数组的排序，效率会高很多。

下面是三向分切算法实现

```java
public class Quick3Way {
    private static int[] waitSortArray = new int[10];
    static{
        for (int i = 0; i < 10; i++) {
//            waitSortArray[i] = (int) (Math.random()*1000);
            waitSortArray = new int[] {10,25,10,36,21,10,89,10,9,8,6,2,100};
        }
    }
    public static void sort(int[] array){
        sort(array, 0, array.length -1);
    }
    // 定义排序的方法体
    public static void sort(int[] array, int lo, int hi){
//        if (hi <= lo+5) { InsertSorted.sort(array, lo, hi); return;} // 将快排算法中遇见的小数组使用插入排序
        if (hi <= lo) return;
        int lt = lo, i = lo+1, gt = hi; int v = array[lo];
        while (i <= gt){
//            int cmp = array[i].compareTo(v);
            if (array[i] < v) exch(array, lt++, i++);
            else if (array[i] > v) exch(array, i, gt--);
            else i ++;
        } // 现在 a[lo .. lt-1] < v = a[lt .. gt] < a[gt+l .. hi]成立
        sort (array, lo, lt - 1);
        sort(array, gt+ 1, hi);
    }
    // 定义交换元素的方法
    public static void exch(int[] array, int i, int j){
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
    // 定义比较方法less
    public static boolean less(int a, int b){
        return a < b;
    }
    // 定义测试方法，main
    public static void main(String[] args) {
        SelectSorted.show(waitSortArray);
        SelectSorted.sort(waitSortArray);
        boolean judge = SelectSorted.isSorted(waitSortArray);
        SelectSorted.show(waitSortArray);
    }
}

```
