---
title: 排序的应用技巧和重要性
author: lijiabao
date: 2021-03-26 17:30
categories: ["DataStruture and Alogrithm"]
tags: ["DataStruture and Alogrithm"]
---

记录一下排序在实际应用中的一些需要了解的技巧和理论，并介绍了排序算法的稳定性和时空复杂度。

## 排序的应用技巧

为了更好的了解排序算法和优先级队列在实际中的应用，下面将会介绍一些典型的应用。希望可以通过这些应用激活灵感，找到合适的应用场景以及合适的算法应用。

排序算法的重要性在于：有序数组中查找某个元素远比在无序数组更简单。

实际的一些排序应用：

- 电话簿按照姓氏排序，查找就变得更容易
- 搜索引擎按照搜索结果的相关性或者重要性来显示搜索结果
- 矩阵处理工具将特征值按照降序排序
- 在一些应用中，排序是一个重要的子问题：数据压缩、计算机图形学、投票等

当数组或者队列有序之后，很多其他任务也会因此变得简单不少。

### 各种数据类型的排序

实现的泛型排序算法是实现了Comparable接口的对象组成的数组，这种机制使得我们可以将任意实现了Comparable接口的数据类型排序。实现Comparable接口只需定义一个comparaTo方法来实现大小关系的比较。对于String、Integer、Double和File以及URL类型的数组排序，因为这些类都实现了Comparable接口

### 交易事务

排序算法的其中一种典型应用就是商业数据处理。比如：公司的每笔交易记录都保存了所有的相关信息，包括客户名、日期、金额等。对于这类数据，排序的选择可以是根据交易时间来排序查看或者根据交易额大小排序查看交易信息。为了可以实现这一中数据类型的比较，实现Comparable接口中的compareTo方法来进行比较。最后使用这个数据类型的泛型排序算法实现排序。因此任何的排序算法，在处理这类交易数据都是可行的。

交易数据类型的实现：

```java
public class Transaction implements Comparable{
    private Date date;
    private Double amount;
    private Person client;
    @Override
    public int compareTo(Transaction that){
        return this.date.comparaTo(that.date);  // 实现日期比较排序
        // return this.amount.compareTo(that.amount);  // 比较交易额
    }
}
HeapSorted<Transaction>.sort(Transaction[] a); // 调用泛型排序算法进行排序
```

### 指针排序

类似上述的交易数据类型处理操作被称之为指针排序，因为只处理元素的引用而不移动数据本身。C和C++中，需要指出操作的是数据还是指向数据的指针，Java中指针操作时隐式的。在Java中，除了基本数据类型之外，方法操作的总是数据引用（指针），而不是数据本身。指针排序添加了一层间接性，因为数组保存的时待排序对象的引用，而不是对象本身。

对于多引用的数组，可以将同一组数据的不同不扽按照多种方式排序（可能需要使用多键）

### 不可变的键

对于排序号的数组或队列，如果还可以修改键值，可能会破坏数组或队列原本的有序性，可能会导致排序的不正常工作。因此，可以使用不可变的数据类型作为键来避免这个问题。Java中可能可以用作键的数据类型：String、Integer、Double和File都是不可变的

### 廉价的交换

使用引用的另外一个好处是，不必要移动整个元素。对于任意大小的元素，使用引用使得在一般情况下交换的成本和比较的成本几乎相同（代价是需要额外空间来存储这些引用）。如果键值很长，交换成本甚至低于比较的成本（普通数字排序算法性能研究就是观察比较和交换的总数，这里是假设比较和交换成本相同）

### Comparator接口应用

#### 多种排序算法

很多时候，我们都希望根据情况将一组对象按照不同的方式进行排序。Java中有一个Comparator接口允许在一个类中实现多种排序方法。这个接口只有一个compare方法来比较两个对象。

Comparator接口允许我们为任意数据类型定义任意多种排序方法。用Comparator接口代替Comparable接口能够更好的将数据类型的定义个两个该类型的对象应该如何比较的定义区分开来。

比较两个对象可以有多个标准，Comparator接口使得我们能够在其中进行选择。

#### 多键数组

一般情况下，元素的多种属性（也就是比较对象的字段）都可能被用作排序的键。就拿最开始提出的交易例子，有时间、交易客户和交易金额三个属性可以用来排序，对于这种多键的灵活实现，可以利用Comparator接口来实现，可以定义多个比较器。

利用后面的比较器的代码来使用排序方法的示例：

```java
// 定义排序方法，选择排序方法
public static void sort(Object[] a, Comparetor c){
    for (int i=0;i<a.length;i++)
        for (int j=i;j>0 && less(c,a[j],a[j-1]);j--)
            exch(a,j,j-1);
}
// 利用所给的比较器比较两个元素之间特定键的大小关系
public static boolean less(Comparator c, Object v, Obeject w){
    return c.compare(v,w) < 0;
}
// 实现交换元素的方法
public static void exch(Object[] a, int i, int j){
    Object temp = a[i];
    a[i] = a[j];
    a[j] = temp;
}
```

#### 使用比较器实现优先队列

比较器的灵活性也可以应用于优先级队列上。可以根据不同的键实现不同的优先级队列，也是十分方便的

- 为实现的队列类添加一个实例变量Comparator和一个构造函数（接受比较器并初始化实例变量的比较器）
- 在less方法中利用比较器来实现比较（这时候需要注意`NullPointerException`问题）因此，做一个非null判断再取执行比较可以增加代码的健壮性。

```java
import java.util.Comparator;

public class Transaction{
    private final String person;
    private final Date date;
    private final Double amount;
    
    // 定义按照人名来比较的比较器
    public static class PersonOrder implements Comparator<Transaction>{
        public int compare(Transaction v, Transaction w){
            return v.person.compareTo(w.person);
        }
    }
    // 定义按照日期来比较的比较器
    public static class DateOrder implements Comparator<Transaction>{
        public int compare(Transaction v, Transaction w){
            return v.date.compareTo(w.date);
        }
    }
    // 定义按照金额来比较的比较器
    public static class AmountOrder implements Comparator<Transaction>{
        public int compare(Transaction v, Transaction w){
            return v.amount.compareTo(w.amount);
        }
    }
}
```

### 稳定性

排序算法的稳定性通过判断是否可以保留数组中重复元素的相对位置。如果可以保证相对位置的不改变，则称之为稳定的排序算法。**就是经历过排序之后，原本有序的部分仍然要保持相对的有序**。

实际例子：考虑一个需要处理大量含有地理位置和时间戳事件的应用程序。首先我们按照事件的事件顺序将其排序号存储到数组。现在需要再按照地理位置切分，一种简单的方法是将数组按照地理位置排序。此时，如果对于地理位置排序时所用的排序算法不稳定，排序后的事件记录可能不是按照事件顺序排序的了

几大排序算法的稳定性和事件空间复杂度如下：

| 排序方法 | 稳定性 | 时间复杂度(平均、最好、最坏)     | 空间复杂度 |
| -------- | ------ | -------------------------------- | ---------- |
| 选择排序 | 不稳定 | `O(n2)、O(n2)、O(n2)`            | O(1)       |
| 插入排序 | 稳定   | `O(n2)、O(n)、O(n2)`             | O(1)       |
| 冒泡排序 | 稳定   | `O(n2)、O(n)、O(n2)`             | O(1)       |
| 希尔排序 | 不稳定 | `O(nlogn)、O(nlog2n)、O(nlog2n)` | O(1)       |
| 归并排序 | 稳定   | `O(nlogn)、O(nlogn)、O(nlogn)`   | O(n)       |
| 快速排序 | 不稳定 | `O(nlogn)、O(nlogn)、O(n2)`      | O(logn)    |
| 堆排序   | 不稳定 | `O(nlogn)、O(nlogn)、O(nlogn)`   | O(1)       |
| 计数排序 | 稳定   | `O(n+k)、O(n+k)、O(n+k)`         | O(k)       |
| 桶排序   | 稳定   | `O(n+k)、O(n+k)、O(n2)`          | O(n+k)     |
| 基数排序 | 稳定   | `O(n✖k)、O(n✖k)、O(n✖k)`         | (n+k)      |

<font color='red'>实际上有很多的方法可以将排序算法变成稳定的，但是一般只在稳定性是必要的时候稳定的排序算法才有有时。实际上达成稳定需要很多的时间和空间才能做到。</font>

## 排序算法的选择

从上面的表格我们可以直观的知道常用的排序算法的时空复杂度以及算法稳定性

> 快速排序算法是最快的通用排序算法
>
> 无数的计算机系统已经证明了这一点。总结来说，<font color='red'>快速排序算法之所以快是因为内循环的指令少，而且快速排序可以利用缓存(因为总是顺序访问数据)。快速排序的运行时间增长数量级cNlogN，其中c比其他线性对数的排序算法要更小。如果使用了三向切分的快速排序实际可能出现某些输入变成了线性级别，其他算法需要线性对数空间</font>

但是在需要稳定性的场景下，归并排序更好。当然后面也出现了像上面表格中的后三种排序算法，他们的时间复杂度耕地。通用的排序算法还是快速排序更好（时间很重要的程序）。

### Java标准库实现的排序

实际上Java系统库中的排序方法`java.util.Arrays.sort()`，传入不同的参数类型，代表了不同的排序算法

- 每种原始数据类型都有一种不同的排序算法
- 一个适用于所有实现了Comparator接口的数据类型的排序算法
- 适用于实现了Comparable接口的排序方法

这个排序算法实际上适用了三向快速排序算法和归并排序算法，还可以自己实现compareTo和compare方法定义自己的比较逻辑。大部分情况已经基本够用