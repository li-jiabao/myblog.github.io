---
title: 散列表
author: lijiabao
date: 2021-04-04 15:30
categories: ["查找", "DataStruture and Alogrithm"]
tags: ["查找", "DataStruture and Alogrithm"]
---

介绍散列表，一种实现字典（键-值）操作的有效数据结构，以及散列表的哈希函数选择以及哈希冲突的解决方案。

## 散列表

散列表最坏情况下，查找一个元素的时间和链表一样为线性级别，但是实际应用中，散列表查找性能是很好的，在一些合理的情况下，散列表查找一个元素的平均时间是常数级别O(1)。

使用散列表的查找算法分成两步：

1. 先利用散列函数将需要查找的键转化为数组的索引，理想情况下，不同的键都能转换为不同的索引值。但是存在两个不同的键散列函数之后得到相同的索引值，这种情况称为散列碰撞（哈希冲突）
2. 哈希冲突的解决：拉链法和线性探测法

散列表是一种典型的空间换时间的例子，如果没有内存限制，可以直接使用键作为数组索引，那么所有查找操作只需要访问一次内存。但是由于键的内存消耗比较大因此不会直接使用键为索引

### 散列函数

对于查找时，首先需要进行的就是键转换为数组的索引。如果数组为M大小，那这个散列函数就需要能够将任意键转换为该数组范围内的索引。这个散列函数应该具有：计算简单，并且可以均匀分布所有的键，对于任意键，0到M-1之间的每一个整数都有相等的可能性与之对应（和键无关）

散列函数和键的类型有关：<font color='red'>对于每种类型的键我们都需要一个与之相对应的散列函数</font>。如果键是一个数值，可以直接使用，如果键是字符串，则需要将字符串转换为一个数（Java中常见的数据类型都有自己散列值的默认实现）

**下面将介绍一些<font color='red'>常用的散列函数：</font>**

1. **除留取余法**：取关键字被某个不大于散列表表长m的数p（p<=m）除后所得的余数作为散列地址。H(key) = key MOD p。这个方法p的选择很重要，一般就取散列表表长或者素数，如果p选的不好，容易产生碰撞。不仅可以对关键字取模，也可以在折叠、平方取中等运算之后取模。
2. **直接寻址法**：取关键字或者关键字的某个线性函数值为散列地址。H(key)=a×key+b，其中a、b为常数，这种散列函数称为自身函数
3. **数字分析法**：就是去发现数字背后的规律，尽可能使用这些数据的规律来构造冲突几率低的散列地址，比如：身份证上的规律，同一个地区的身份证前面的数字基本都相同，但是身份证的后面几位差别很大，因此可以利用身份证后面几位来构造散列地址
4. **平方取中法**：取关键字平方后的中间几位作为散列地址
5. **折叠法**：将关键字分割称位数相同的几部分，然后取这及部分作的叠加和作为散列地址
6. **随机数法**：选择一个随机函数，取关键字作为随机函数的种子生成随机值作为散列地址，通常用于关键字长度不同的场合

#### 整数

将整数散列最常用的方法是除留取余法。选择大小为素数M的数组，对于任意正整数k，计算k除以M的余数。取余方法可以很好的将键散步在0-M-1的范围内。但是如果M不是素数，可能无法利用键中包含的所有信息，可能导致我们没办法均匀的散列键值。

#### 浮点数

如果键是0-1之间的实数，可以将其乘以M并四舍五入得到一个0到M-1的索引、但是这个方法有缺陷，就是键的高位起的作用更大，最低为对散列结果没啥影响。修正这个问题的办法是将键表示为二进制数然后使用除留取余法

#### 字符串

字符串只需要将字符串转换成大的整数即可，然后利用除留取余法计算字符串进行散列值求解

```java
int hash = 0;
for (int i=0;i<s.length;i++)
    hash = (R * hash + s.charAt(i)) % M;
// 只要R够小，不会造成溢出即可
```

#### 组合键

如果键中包含多种类型，可以使用和字符串一样的操作将他们混合，例如Date类型的键，可以按照下列的操作进行：

```java
int hash = ((( dat*R + month ) % M ) * R +year ) % M;
```

只要R足够小不造成溢出，也可以得到一个0到M-1之间的散列值

#### Java中和哈希相关内容

Java中让所有数据类型都继承了一个能够返回一个32位整数的hashCode()方法。而且Java中规定：equals和hashCode的判断必须保持一致性（equals方法相等的两个对象，hashCode也应该相等）。如果需要重写hashCode或者equals，这两个方法都需要重写。默认的散列函数只返回对象的内存地址，但是这只适用于很少的情况。

Java中位很多常用的护具类型重写了hashCode方法：Integer、URL、File、Double和String等

##### hashCode值转化为数组索引

实际散列表实现查找时，需要的时数组的索引而不是32位的整数，因此会使用hash方法来获取键对应的索引值：

```java
private int hash(Key key)
{
    return (key.hashCode() & 0x7fffffff ) % M;
}
// 上面这段代码会将符号位屏蔽（32位整数变为一个31位非负整数），然后使用除留取余法计算余数获取索引
// 为了更好利用原有散列值的所有位，会选取M为素数
```

##### 自定义hashCode方法

对于hashCode方法，更希望的是它能够将键平均的散布为所有可能的32位整数。对于任意的对象Obj，你都可以使用`Obj.hashCode()`并认为有均等的机会得到2的32次方中任意一个32位整数值。Java常用的数据类型都可以实现这一点。对于自定义的数据类型，我们需要自己来实现这个hashCode方法。下面是一个实现hashCode的例子：

```java
public class Transaction
{
    private final String who;
    private final Date when;
    private final double amount;
    // 重写，或者说叫做自定义hashCode方法
    public int hashCode()
    {
        int hash = 17;
        hash = 31 * hash + who.hashCode();
        hash = 31* hash + when.hashCode();
        hash = 31 * hash + ((Double) amount).hashCode();
        return hash;
    }
    // 这是自定义hashCode方法一种比较简单的实现方案
}
```

##### 软缓存

如果散列值的计算很耗时，可以将每个键的散列值缓存起来。可以通过在每个键中使用一个hash变量来保存它的hashCode的返回值，第一次调用hashCode方法时，我们需要计算该对象的散列值，但之后对hashCode方法的调用会直接返回hash变量的值。Java的String对象就使用这种操作

```java
public class Transaction
{
    private final String who;
    private final Date when;
    private final double amount;
    private final Integer hash = null;
    // 重写，或者说叫做自定义hashCode方法
    public int hashCode()
    {
        if (this.hash == null)
        {
            int hash = 17;
            hash = 31 * hash + who.hashCode();
            hash = 31* hash + when.hashCode();
            hash = 31 * hash + ((Double) amount).hashCode();
            this.hash = hash;
            return hash;
        }
        return this.hash.intValue();
    }
    // 实现软缓存的方法，比较简单粗糙的实现
    // 这种方法可以减少计算量，也可以较为有效的改善查找的算法效率
}
```

<font color='red'>想要实现一个优秀的散列方法需要满足三个条件：</font>

- 一致性：等价的键必然产生相等的散列值
- 高效性：计算简便，时间效率高
- 均匀性：可以均匀的散列所有的键

Java中专家们已经很好的实现了hashCode方法，因此在使用Java时只需要调用hashCode即可。

有性能要求时应该谨慎的使用散列，因为糟糕的散列函数常常是性能问题的元凶，可以工作但是效率低。

保证均匀性最好的办法就是保证键的每一位都在散列值的计算中起到了相同的作用，最常见的散列函数错误就是忽略了键的高位。

<font color='red'>使用散列函数时，性能比较重要的时候应该测试使用的散列函数：计算散列函数和比较两个键，哪个耗时更多？散列函数是否可以将一组键均匀的散步到0和M-1之间？</font>

### 散列冲突解决

散列冲突：也就是哈希碰撞，当两个或者多个键的值相等的情况。对于这种情况，需要采取一些特殊的方法来进行哈希冲突的解决，下面将介绍一下哈希冲突的几种常用解决方法

**<font color='red'>常用的处理散列冲突的方法：</font>**

1. 开放寻址法：
2. 再散列法：
3. 链地址法（拉链法）：
4. 建立一个公共溢出区：

#### 拉链法

比较直接的一种方法，就是将大小位M的数组中的每个元素指向一条链表，链表中的每个结点都存储了散列值为该元素的索引的键值对，这种方法称为**拉链法**。

![拉链法示意图](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E6%8B%89%E9%93%BE%E6%B3%95%E7%9A%84%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

一种比较简单的方法（效率较低）：通过创建大小为M的数组，数组的元素是链表。当键值散列后找到自己的索引位置之后，将键值对应的键值对添加到链表中，如果碰见了哈希冲突，就将元素继续添加到这个索引处的链表上。这种方法可以较好的解决哈希冲突

因为是使用M条链表保存N个键，无论键在各个链表的分布如何最终链表的平均长度为N/M。

```java
/**
* 这是基于之前实现的顺序查找链表来实现的拉链法的散列表
*/
public class SeparateChainingHashST<Key extends Comparable<Key>, Value>
{
    private int N;  // 键的数量
    private int M;  // 散列表的大小
    private SequentialSearchST<Key, Value>[] st;
    // 无参构造方法
    public SeparateChainingHashST()
    {
        this(997);
        // 默认构造函数使用了997条链表，对于较大的符号表，这种实现会比原始的顺序查找链表快1000倍
        // 当你可以预测符号表大小时，这种实现可以获取不错的性能
    }
    // 指定散列表容量的构造方法
    public SeparateChainingHashST(int capcacity)
    {
        this.M = capacity;
        // Java中不允许泛型数组，需要对数组声明时进行类型转换
        st = (SequentialSearchST<Key, Value>[]) new SequentialSearchST[M];
        for (int i=0;i<st.length;i++)
        {
            st[i] = new SequentialSearchST();
        }
    }
    // 获取键值对应的hashCode值对应的数组索引
    public int hash(Key key)
    {
        return (key.hashCode() & 0x7fffffff) % M;
    }
    // 查找操作
    public Value get(Key key)
    {
        return (Value) st[hash(key)].get(key);
    }
    // 插入操作
    public void put(Key key,Value value)
    {
        st[hash(key)].put(key, value);
    }
}
```

尽管上述的代码相比顺序查找链表实现的符号表已经可以获取较为不错的性能，但是因为需要提前定义好数组的大小。

另外一种更可靠的方案时动态调整链表数组的大小，这样无论在符号表中有多少键值对都可以保证链表较短。

##### 性能分析

如果使用拉链法时，你可以准确的估计所需要的散列表的大小，调整数组（超过M/2，数组长度变为2倍，小于M/8，数组大小缩减一半）的工作不是必须的，只需要根据查找耗时和（1+N/M）成正比的规律选取一个合适的M即可。但是相比之下，下面介绍的线性探测法，调整数组大小是必须的，<font color='red'>因为如果插入的键值对数量超过预期时查找时间会变得非常长，还会在散列表被填满时，进入无限循环</font>

#### 线性探测（开放寻址法）

实现散列表解决哈希冲突的另外一种实现就是使用大小为M的数组保存N个键值对（M>N）。需要依据数组中的空位来解决哈希碰撞。这种策略的所有方法称为开发地址法散列表

最简单的一种开放寻址法散列表的实现方法：**线性探测法**

> 碰撞发生时，直接检查散列表的下一个位置（索引值加一）

但是这样的实现方法可能会产生三种结果：

- 命中，该位置的键和被查找的键相同
- 未命中，键为空（该位置没有键）
- 继续查找，该位置的键和被查找的键不同

```java
public class LinearProbingHashST<Key extends Comparable<Key>, Value>
{
    private int M = 16;  // 构建散列表的大小
    private int N;  // 散列表中键的数量
    private Key[] keys;
    private Value[] values;
    // 创建构造方法
    public LinearProbingHashST()
    {
        keys = (Key[]) Object[M];
        values = (Value[]) Object[M];
    }
    public LinearProbingHashST(int size)
    {
        keys = (Key[]) Object[size];
        values = (Value[]) Object[size];
    }
    // 插入元素方法
    public void put(Key key, Value value)
    {
        // 首先判断数组中的元素是否已经超过散列表一半，超过了就改变数组大小
        if (N>=M/2) resize(2*M);
        int hash = hash(key);
        for (hash;keys[hash]!=null;hash=(hash+1) % M)
        {
            if (keys[hash]==key) { values[hash]=value;return;}
        }
        keys[hash] = key;
        values[hash] = value;
        N++;
    }
    // 查找元素方法
    public Value get(Key key)
    {
        for (int hash = hash(key); keys[hash] != null; hash = (hash+i) % M)
        {
            if (keys[hash].equals(key)) return values[hash];
        }
        return null;
    }
    // 改变数组大小，保证数组元素一直保持在数组的[1/8, 1/2]区间，可以获取较好的性能
    private void resize(int size)
    {
        LinearProbingHashST<Key, Value> lpt = new LinearProbingHashST(size);
        for (int i =0; i < M; i++)
        {
            if (keys[i]!=null)
            	lpt.put(keys[i], values[i]);
        }
        this.keys = lpt.kesy;
        this.values = lpt.values;
        this.M = lpt.M;
    }
    // 获取hashCode对应的数组索引：
    public int hash(Key key)
    {
        return (key.hashCode & 0x7fffffff) % M;
    }
    // 删除操作，当数组的大小变为1/8的时候，需要进行数组尺寸减半的操作
    public void Value delete(Key key)
    {
        if (!contains(key)) return;
        int hash = hash(key);
        while (!key.equals(keys[hash]))
            hash = (hash + 1) % M;
        keys[hash] = null;
        values[hash] = null;
        hash = (hash + 1) % M;
        // 进行下面操作的原因是防止删除元素出现的null使得元素遍历在这儿断了，导致没法获取后面的元素
        while (keys[hash] != null)
        {
            Key tempKey = keys[hash];
            Value tempValue = values[hash];
            keys[hash] = null;
            values[hash] = null;
            N--;
            put(tempKey, tempValue);
            hash = (hash + 1) % M;
        }
        N--;
        if (N>0 && N <= M/8) resize(M/2);
    }
}
```

##### 性能分析

开放寻址法的散列表的性能和拉链法一样，依赖于N/M的比值，在这里这个比值被称为散列表的使用率，而在拉链法中，这个值是每条链表的长度（大于1），在线性探测方法中，这个值不可能大于1，在该方法的实现中，并不允许N/M达到1，这样会导致无限循环。为了保证性能，数组的使用率要保证在1/8和1/2之间。

> 线性探测法在散列表快满的时候查找所需要的探测次数是巨大的（N/M越趋近于1，探测次数也越多），使用率小于1/2时探测次数较少。因此线性探测法实现必须要可以动态调整数组，动态调调整数组的方法在上述代码中已经实现。利用新键一个线性探测表来实现数组大小改变和元素重新插入



#### 调整数组大小

对于散列表性能优化中使用的数组调整的范围是[1/8, 1/2]之间。

如果只是为了实现数组大小的动态调整，范围可以是[1/4, 1/2]之间

散列表中的动态调整实现，应该在插入元素的时候首先进行判断数组大小是否已经超过1/2，超过了就进行数组大小翻倍的操作。然后在删除元素操作完成之后，进行判断，元素是否已经少于数组大小的1/8，满足就对数组大小进行减半。

##### 拉链法和线性探测法

对于拉链法来说，调整数组大小并不是必须的，如果可以准确的估计大小，就可以不用动态调整，只需要根据查找效率和（1+N/M）成正比的结论来选择合适的M即可。

对于线性探测法来说，调整数组大小是必须的，如果插入的键值对数量超过预期查找时间会变得很长，而且数组满的话会进入无限循环。

##### 均摊分析

当动态调整数组大小，需要找出均摊成本的上限，因为散列表长度加倍的插入操作需要大量的探测。

> 假设散列表可以自己调整数组大小，初始为空。执行任意顺序的t次查找、插入和删除操作所需要的时间和t成正比，所使用的内存量总是在表中的键的总数的常数因子范围内



### 内存使用

![各种符号表实现的内存使用](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E5%90%84%E7%A7%8D%E7%AC%A6%E5%8F%B7%E8%A1%A8%E5%AE%9E%E7%8E%B0%E7%9A%84%E5%86%85%E5%AD%98%E4%BD%BF%E7%94%A8.png)

实际运用中：拉链法为每一个键都分配了内存，线性探测法则是使用了两个很大的数组

对于不同的实现的内存管理要求也不相同，需要专家来进行。



### 总结

散列表能够支持和数组大小无关的常数级别的插入和查找操作是可能的，这也是符号表期望的最优性能

但是哈希表也有缺点：

- 每个类型的键都需要一个优秀的散列函数
- 性能取决于散列函数的质量
- 散列函数的计算可能复杂且昂贵
- 符号表难以支持有序性相关的操作（拉链法实现的散列表的有序性相关操作运行时间基本都是线性的）

在对于顺序并不重要的时候，拉链法实现的符号表是一种最快的方法。可以提供较为快速的插入查找操作。