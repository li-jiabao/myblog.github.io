---

title: 查找算法绪论--符号表
author: lijiabao
date: 2021-03-31 23:30
categories: ["查找", "DataStruture and Alogrithm"]
tags: ["查找", "DataStruture and Alogrithm"]
---

介绍查找算法的二分查找，符号表的定义和常见的几种实现，并列举了符号表各种实现的一些优缺点

## 符号表

符号表是一张抽象的表格，将信息存储在这个符号表，按照指定的键来搜索并获取这些信息，键和值的具体意义根据不同应用场景而改变。符号表中保存了很多的键和信息，实现高效的符号表很重要。

符号表有时被叫做字典：类似字典，键就是单词，值就是单词对应的定义解释等内容。

符号表有时还被叫做索引：书本中按照字母顺序或者姓名排序的部分，键就内容相关的关键字，值就是这部分内容所在的位置。

查找算法中会主要包含三种经典数据类型实现的高效符号表：二叉查找树、红黑树和散列表。

符号表就是将一个键和一个值对应联系起来，希望可以通过键查找来获取键对应的值。符号表构建需要首先定义其背后的数据结构，并指明创建并操作这种数据结构实现插入、查找等操作所需要的算法。

最主要的两个操作：插入（put）和查找（get）

### 符号表API

`public class ST<Key,Value> implements Iterable<Key>`类

| 接口API                               | 描述                                     |
| ------------------------------------- | ---------------------------------------- |
| public ST()                           | 创建一个符号表                           |
| public void put(Key key, Value value) | 将键值对存入表中（值为空将键从表中删除） |
| public void get(Key key)              | 获取键对应的值                           |
| public void delete(Key key)           | 从表中删除键及其对应的值                 |
| public boolean contains(Key key)      | 查看表中是否存在该键                     |
| public boolean isEmpty()              | 判断表是否为空                           |
| public int size()                     | 表的键值对数目                           |
| public Iterable<Key> keys()           | 表中所有键的集合                         |
| public Iterable<Key> values()         | 表中所有值的集合                         |

对于符号表设计时，建议使用泛型来定义键值对的类型，使得代码更加灵活多用，并且可以用来很好的区分键值对

对于键的设计，需要遵循以下规则：

- 每个键对应一个值，表中不允许存在重复键
- 如果已经存在这个键，后来的值会替代先前存在的值
- 键不能有null，值也不允许有null，因为null会导致Java错误
- 键的等价性：对于键的等价性判断可以重写equals来实现不一样的表示。对于键最好使用不可变的数据类型作为键，不然无法保证键的一致性。

删除操作的实现方法：

- 延时删除：先将需要删除的键对应的值设置为null，后面再统一删除值为null的键值对。`put(Key key, Value value)`
- 即时删除：立刻从表中删除键值对

迭代：为了方便的处理获取表中的所有键值，会继承实现接口`Iterable<Key>`，强制实现代码需要实现iterator方法来返回一个实现了hasNext和next方法的迭代器

### 有序符号表

典型应用程序中，键都是Comparable的对象，因此可以使用comparaTo方法来比较两个键。

许多的符号表实现都是利用Comparable接口带来的键的有序性来更好的实现put和get方法。当实现了符号表的有序之后，就可以更好的扩展它的API。下面表中是有序符号表定义的API

`public class ST<Key extends Comparable<Key> key, Values value>`类



| 接口API                                       | 描述                                     |
| --------------------------------------------- | ---------------------------------------- |
| public ST()                                   | 创建一个符号表                           |
| public void put(Key key, Value value)         | 将键值对存入表中（值为空将键从表中删除） |
| public void get(Key key)                      | 获取键对应的值                           |
| public void delete(Key key)                   | 从表中删除键及其对应的值                 |
| public boolean contains(Key key)              | 查看表中是否存在该键                     |
| public boolean isEmpty()                      | 判断表是否为空                           |
| public int size()                             | 表的键值对数目                           |
| public Iterable<Key> keys()                   | 表中所有键的集合                         |
| public Iterable<Value> values()               | 表中所有值的集合                         |
| public Key min()                              | 获取表中最小键                           |
| public Key max()                              | 获取表中最大元素                         |
| public Key floor(Key key)                     | 小于等于key的最大键                      |
| public Key ceiling(Key key)                   | 大于等于key的最大键                      |
| public int rank(Key key)                      | key的排名                                |
| public Key select(int k)                      | 排名为k的键                              |
| public void delMin()                          | 删除最小的键                             |
| public void delMax()                          | 删除最大键                               |
| public int size(Key lo, Key hi)               | 位于键lo和hi之间元素数量                 |
| public Iterable<Key> keys(Key lo, Key hi)     | 位于lo和hi之间的所有键                   |
| public Iterable<Value> values(Key lo, Key hi) | 位于lo和hi键之间的所有值                 |

**对于异常情况**：当一个方法中需要返回一个键但是表中没有合适的键可以返回时，约定需要抛出异常（或者返回null）对于上述API：min、max、floor、ceiling、delMin和delMax都需要抛出异常。对于select方法，若是k值超出了size范围，也需要抛出异常。

**键的等价性**：对于键的等价性在Java中最佳实践就是维护所有Comparable类型中的comparaTo和equals方法的一致性。对于任意两个Comparable类型一定要满足`a.ComparaTo(b) == 0`和`a.equals(b)`的返回值要相同。

>  对于符号表的查找的成本模型：
>
> 统计比较的次数（等价性测试或者键的相互比较）内循环不进行比较（较少）的情况，我们会统计数组的访问次数

### 有序符号表的实例

简单测试用例：

```java
public class SimpleTest{
    public static void main(String[] args){
        var st = new ST<String, Integer>(); // 新建一个符号表
        for (int i=0;i<10;i++){
            st.put("A"+i, i);  // 向符号表中插入元素
        }
        for (String s:st.keys()){
            // 遍历打印符号表键值对
            System.out.println(s + ": " + st.get(s));
        }
    }
}
```

符号表的统计单词出现次数的用例：

```java
public class WordCounter{
    public static void main(String[] args){
        var st = new ST<String, Integer>(); // 新建一个符号表
        while (!StdIn.isEmpty()){
            String word = StdIn.readString(); // 读取输入流中的字符串单词
            if (!st.contains(word)) st.put(word, 1); // 判断符号表是否存在键，不存在就新增
            else st.put(word, st.get(word)+1); // 存在就在原来键对应值加一
        }
        // 找出出现频率最高的键值对
        String max = " ";
        st.put(max, 0);
        for (String s:st.keys()) // 遍历所有的键，获取键对应的出现次数，找到出现次数最大的键和值
            if (st.get(s)>st.get(max)) max=s;
        System.out.println(max+": "+st.get(max));
    }
}
```

符号表在实际应用中的一些特性：

- 混合使用查找和插入的操作
- 大量的不同键
- 查找操作比插入操作多得多
- 虽然不可预测，但是查找和插入操作的使用模式并非随机

### 无序链表中的顺序查找

无序链表实现的符号表的操作主要是两个：一个是插入put，一个是查找get。前者主要通过遍历链表，来找到需要插入的键的位置，如果找到键，就对该键对应的值进行更新，如果没有找到，就插入该键到链表的开头，并赋值给到这个键。后者主要是遍历链表，适用equals方法进行键的比对，相等就返回该键对应的值，如果没有匹配就返回null

上述方法就成为顺序查找：查找过程一个一个键去遍历搜索，利用equals来判断是否相等来看是否匹配

> 无序链表实现的符号表中，未命中的查找和插入操作都需要N次比较，命中的查找最坏情况需要N次比较
>
> 向一个空链表中插入N个不同的键需要N^2^/2次比较

### 有序数组符号表的二分查找

利用两个有序数组进行符号表的创建，并使用二分查找方法来进行快速的查找和插入。

```java
public class ArrayST<Key extends Comparable<Key>,Value>
{
    private Keys[] keys;
    private Values[] values;
    private int N;
    // 初始化数组符号表的容量
    public ArrayST(int capacity)
    {
        keys = (Keys[]) new Comparable[capacity];
        values = (Values[]) new Comparable[capacity];
    }
    public int size()
    {
        return N
    }
    // 利用二分查找，找出需要插入键所在的位置，然后插入到该位置
    public put(Key key, Value value)
    {
        int rank = rank(key);
        if (rank<N && key.compareTo(keys[rank])==0)
            values[rank] = value; return;
        for (int i=N;j>i;j--)
        { keys[i]=keys[i-1]; values[i]=values[i-1]}
        keys[i]=key;
        values[i]=value;
        N++;
            
    }
    // 查找键值对的算法实现，利用二分查找
    public Value get(Key key)
    {
        if (N==0) return null;
        int rank = rank(key);
        if (rank<N && keys[rank].compreTo(key)) return values[rank];
        else
            return null;
    }
    // 基于二分查找实现的查询元素位置的算法
    public int rank(Key key)
    {
        int lo=0,hi=N-1;
        while (lo<=hi)
        {
            // 进行二分操作
            int mid = lo + (hi-lo)/2;
            // 对键进行比较，查看与当前二分位置键的比较情况
            int comp = key.compareTo(keys[mid]);
            if (comp > 0) lo = mid+1;
            else if (comp < 0) hi = mid-1;
            // 相等就返回二分位置
            else return mid;
        }
        // 循环完毕未发现与之相等的元素，因此返回该键的位置
        return lo;
    }
}
```

> 对于N个数据的数组的二分查找需要最多比较次数（lgN+1)，不管是否成功找到。插入和删除操作最差还是需要线性级别N次比较。
>
> 对于大小为N的有序数组插入一个新的元素需要访问2N次数组，对于有序数组实现的空符号表插入N个元素最差需要访问N\*N次数组

<font color='red'>尽管对于二分查找实现的符号表，查找的时间效率已经提高到了对数级别，但是它的插入和删除操作仍然还需要线性级别，为了更进一步优化时间效率，应该相办法实现插入和删除的对数级别时间效率。</font>

实际上，达成这种操作是可以的，我们可以采用一种新的数据结构来实现对数级别的插入删除和查找操作，可以结合链式结构来做到，也就是<font color='red'>数组和链表</font>的结合。



### 多种符号表实现优缺点

| 使用的数据结构     | 实现                                     | 优点                                                         | 缺点                                                         |
| ------------------ | ---------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 链表的顺序查找     | SequtialSearchST                         | 适用于小型问题的查找，实现比较简单                           | 对于大型符号表，速度很慢                                     |
| 数组的二分查找实现 | BinarySearchST                           | 数据的查找效率很高，对数级别，空间效率也高，能进行有序性相关操作 | 对于数据的插入和删除，效率较低，平均为线性级别的时间复杂度   |
| 二叉查找树         | BST                                      | 结合了二分查找和链表的优点，查找插入效率都很高，对数级别，能够进行有序性的操作，构建相对简单 | 没有性能上届保证，链接需要额外空间                           |
| 平衡二叉树         | RedBlackBST                              | 最优的查找和插入效率，能够进行有序性相关的操作               | 链接需要额外的空间                                           |
| 散列表             | SeparateChainHashST和LinearProbingHashST | 能够快速查找和插入常见类型的数据                             | 需要计算每种类型数据的散列，无法进行有序性相关操作，链接和空结点需要额外的空间 |

