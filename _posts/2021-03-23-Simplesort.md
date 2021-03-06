---
title: 初级排序算法
author: lijiabao
date: 2021-03-23 15:30
categories: ["DataStruture and Alogrithm"]
tags: ["DataStruture and Alogrithm"]
---

介绍一下排序算法的模板和常用的两种初级排序算法和插入排序的一种变体算法--希尔排序

## 初级排序算法

排序算法使用的模板代码：

```java
/**
 * 这里主要是展示排序算法的基本框架结构
 */

public class SortExample {

    // main方法是用来测试算法是否准确,用来测试算法可行性的部分
    public static void main(String[] args) {
        // 在这里写测试算法的代码
    }
    // 对数组元素进行排序
    public static void sort(Comparable[] a){
        // 在这里写排序算法
    }
    // 判断两个课比较元素之间谁更大
    public static boolean less(Comparable v, Comparable w){
        return v.compareTo(w) < 0;
    }
    // 交换数组元素
    public static void exch(Comparable[] a, int i, int j){
        Comparable t = a[i];
        a[i] = a[j];
        a[j] = t;
    }
    // 展示数组的元素,按照顺序一一打印
    public static void show(Comparable[] a){
        for (Comparable i : a){
            System.out.println(i);
        }
    }
    // 判断数组是否已经有序
    public static boolean isSorted(Comparable[] a){
        for (int i=0; i < a.length; i++){
            if ( ! less(a[i], a[i+1]) ) return false;
        }
        return true;
    }
}

```

### 选择排序

选择排序：首先找到数组中最小的元素，将它和数组第一个元素交换位置，然后在剩下的元素找到最小数组，然后将其和第二个元素交换，如此依次执行交换知道数组有序。

选择排序的内循环只是在比较当前元素和目前已知的最小元素（以及索引加一是否越界）每次交换就可以排好一个元素，交换的总次数是N，算法效率和数组大小有关（取决于比较次数）。

选择排序的两个特点：

- **运行时间和输入无关：选择排序已经有序的数组和随机排列数组的排序时间一样长。**其他算法更加善于利用输入的初始状态
- **数据移动是最少的：选择排序用了N次交换，交换次数和数组大小是线性关系**

算法实现的时候逻辑关系一定要找到最好的，减少循环次数，减少操作次数

**选择排序的关键在于每一次循环的时候，找到最小的元素，然后将其与当前循环头的元素交换。**

每次循环之后，下一次循环的体量就会减少一个。

```java
public class SelectSorted {
    private static int[] waitSortArray = new int[10000];
    static{
        for (int i = 0; i < 10000; i++) {
            waitSortArray[i] = (int) (Math.random()*1000);
        }
    }
    // 定义排序的方法体
    public static void sort(int[] array){
        for (int i = 0; i < array.length; i++) {
            int min = i;
            for (int j = i+1; j < array.length; j++) {
                if (less(array,j,min)) min=j;
            }
            exch(array,i,min);
        }
    }
    // 定义交换元素的方法
    public static void exch(int[] array, int i, int j){
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
    // 定义比较方法less
    public static boolean less(int[] array, int a, int b){
        return array[a] < array[b];
    }
    // 定义遍历数组的方法
    public static void show(int[] array){
        for (int i:array){
            System.out.print(i + " ");
        }
        System.out.println();
    }
    // 定义判断是否排序好的方法
    public static boolean isSorted(int[] array){
        for (int i=0;i<(array.length-1);i++){
            if (!less(array, i, i+1)){
                return false;
            }
        }
        return true;
    }

    // 定义测试方法，main
    public static void main(String[] args) {
        InsertSorted.show(waitSortArray);
        TimeCounter time = new TimeCounter();
        InsertSorted.sort1(waitSortArray);
        System.out.println(time.stopTimeCounter());
        boolean judge = InsertSorted.isSorted(waitSortArray);
        InsertSorted.show(waitSortArray);
    }
}

```



### 插入排序

就是参考整理牌的顺序来实现的一种算法：遍历数组，找到当前元素应该所在的插入位置将其插入，为了腾出空间，需要右移这个元素后面的元素。

和选择排序一样，当前索引左边的数组都是有序的，但是最终的位置还没有确定，为了给更小的元素腾位置，需要移动更大的元素。**当索引到达最右边时，排序结束**

插入排序的所需要的时间和输入的元素的初始顺序有关：**已经接近有序的数组比随机顺序的数组排序快得多**

该算法的关键在于每次插入之后，后面的元素需要向后移动一位。起始就是每次外圈循环到的元素和左边排序好的元素进行比较确定插入位置

- 从循环到的元素开始一步一步向前比较，比较如果小就换位（可以将判断放到循环中）
- 从头向循环到的元素循环比较，比较如果小，这个就是插入位置，再进行换位（这种方法效率较上一种效率低）

想要大幅度提升插入排序的速度并不难，只需要再内循环中将较大的元素都向右移动而不总是交换两个元素（访问数组的次数就能减半）

插入排序对于部分有序的数组十分高效，也很适合小规模数组。

```java
public class InsertSorted {
    private static int[] waitSortArray = new int[10000];
    static{
        for (int i = 0; i < 10000; i++) {
            waitSortArray[i] = (int) (Math.random()*1000);
        }
    }
    // 定义排序的方法体
    public static void sort1(int[] array){
        for (int i = 1; i < array.length; i++) {
            for (int j=0;j<i;j++){
                if (less(array,i,j)){
                    insert(array,j,i);
                }
            }
        }
    }
    // 书上的排序
    public static void sort(int[] array){
        for (int i = 0; i < array.length; i++) {
            for (int j=i;j>0 && less(array,j,j-1);j--){
                exch(array,j,j-1);
            }
        }
    }
    // 定义插入方法，并将插入元素之后的元素都往后移动一位
    public static void insert(int[] array, int firstPos, int lastPos){
        int temp = array[lastPos];
        // 自己写的这种算法由于多了一层循环，因此运算的速度比书上代码慢了将近一半
        for (int i=lastPos;i>firstPos;i--){
            array[i] = array[i-1];
        }
        // 因此可以减少循环，看是否可以将条件判断和操作全部糅合在一次循环当中
        array[firstPos] = temp;
    }
    // 定义交换元素的方法
    public static void exch(int[] array, int i, int j){
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
    // 定义比较方法less
    public static boolean less(int[] array, int a, int b){
        return array[a] < array[b];
    }
    // 定义遍历数组的方法
    public static void show(int[] array){
        for (int i:array){
            System.out.print(i + " ");
        }
        System.out.println();
    }
    // 定义判断是否排序好的方法
    public static boolean isSorted(int[] array){
        for (int i=0;i<(array.length-1);i++){
            if (!less(array, i, i+1)){
                return false;
            }
        }
        return true;
    }

    // 定义测试方法，main
    public static void main(String[] args) {
        InsertSorted.show(waitSortArray);
        Timer time = new Timer();
        InsertSorted.sort1(waitSortArray);
        long usedTime = time.stopTimer();
        System.out.println("排序耗时"+usedTime+"ms");
        boolean judge = InsertSorted.isSorted(waitSortArray);
        InsertSorted.show(waitSortArray);
    }
}
```



### 希尔排序

对于大规模乱序数组插入排序很慢，因为只会交换相邻的元素，元素只能一点点的从数组的一段移动到另一端

希尔排序是基于插入排序的快速的排序算法，为了加快速度在插入排序的基础上进行改进，通过交换不相邻的元素来对数组的局部进行排序。希尔排序的思想是使数组中任意间隔为h的元素都是有序的，成为h有序数组。对于h数组，当h为1时，数组就能够排序好。

实现希尔算法一种方法时对于每一个h，使用插入排序将h个子数组独立排序。因为子数组相互独立，更简单的方法是在子数组中将每个元素交换到比它大的元素之前去（比它大的元素右移一格）只需要在插入排序的代码中将移动元素的距离变为h即可。就相当于实现了一个不同增量的插入排序方法。

对于h序列的的选择问题；关键在于序列的选择，算法第四版书中实现的算法使用的1/2（3^k^-1)序列，从N/3开始递减到1，这个序列称为递增序列

对于递增序列的选择，很重要。希尔算法的性能不仅取决于h，还取决于h之间的数学性质如公因子。好的复杂的递增序列性能会好于算法示例所用的序列，但是所用的序列简单性能接近复杂的序列。

希尔排序也可以用于大型数组，对任意排序的数组表现也很好。希尔排序可以解决一些初级排序算法无法解决的问题。

> 通过提升速度来解决其他方式无法解决的问题是研究算法的设计和性能的主要原因之一

对于中等大小的数组希尔算法的运算时间是可以接受的，代码量很小，且不需要使用额外的内存空间

当你需要解决一个排序问题而又没有系统排序函数可用，可以尝试这先用希尔排序，然后在考虑是否值得将他替换为更加复杂的排序算法。

```java
public class ShellSorted {
    private static int[] waitSortArray = new int[10];
    static{
        for (int i = 0; i < 10; i++) {
            waitSortArray[i] = (int) (Math.random()*1000);
        }
    }
    // 定义排序的方法体
    public static void sort(int[] array){
        int N = array.length;
        int h = 1;
        while (h < N/3) h = h * 3 - 1;
        while (h >= 1){
            for (int i=0;i<N;i++){
                for (int j=i;j>h && less(array,j,j-h);j-=h){
                    exch(array,j, j-h);
                }
            }
            h /= 3;
        }
    }
    // 定义交换元素的方法
    public static void exch(int[] array, int i, int j){
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }
    // 定义比较方法less
    public static boolean less(int[] array, int a, int b){
        return array[a] < array[b];
    }
    // 定义遍历数组的方法
    public static void show(int[] array){
        for (int i:array){
            System.out.print(i + " ");
        }
        System.out.println();
    }
    // 定义判断是否排序好的方法
    public static boolean isSorted(int[] array){
        for (int i=0;i<(array.length-1);i++){
            if (!less(array, i, i+1)){
                return false;
            }
        }
        return true;
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

总结

不同的排序算法有不同的适用场景和输入规模。

- 对于部分有序和小规模的数组应该选择插入排序
- 中等体量的算法，可以考虑希尔算法，可以提供较为不错的性能