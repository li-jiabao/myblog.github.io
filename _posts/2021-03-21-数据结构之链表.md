---
title: 数据结构之链表
author: lijiabao
date: 2021-03-21 20:30
categories: ["DataStruture and Alogrithm"]
tags: ["DataStruture and Alogrithm"]
---

## 链表

链表是一种递归的数据结构，或者为空（null）或者是指向一个结点的引用，该节点含有一个泛型的元素和一个指向另一条链表的引用。

结点是一个可能含有任意类型数据的抽象实体，他所包含的指向结点的应用显示了他在构造链表之中的作用

链表的优点

- 可以处理任意类型的数据
- 所需的空间总是和集合大小成正比
- 操作所需的时间总是和集合的大小无关

### 结点记录

面对对象编程中实现链表，使用嵌套类来定义结点的抽象数据类型：

```java
private class Node{
    Item item;
    Node node;
}
```

一般会在需要使用Node类中的类定义并将其标记为private,因为不是为用例准备的.Item是一个占位符,表示为我们希望用链表处理的任意数据类型(可以使用Java的泛型表示任意引用类型)Node类型的实例变量显示了这种数据结构的链式本质：如果first是一个指向Node对象的变量,这种类型的类有时称为记录

### 构建链表

首先需要为每个元素创造一个结点:

```java
Node first = new Node();
Node second = new Node();
Node third = new Node();

// 并为每个结点的item设置所需要的值
first.item = "to";
second.item = "be";
third.item = "or";

// 然后设置next来构造链表
first.next = second;
second.next = third;
// third.next仍然是null,也就是对象创建时初始化的值,链接就是表示对结点的引用
```

![链接构建列表](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E9%93%BE%E6%8E%A5%E6%9E%84%E5%BB%BA%E5%88%97%E8%A1%A8.png)

#### 表头插入结点

向链表中插入新的结点最容易做到的就是链表的开头.首先将first结点保存到oldfirst,然后新建一个结点赋予first并将item设置为新值,next设置为oldfirst

```java
Node oldfirst = first;
first = new Node();
first.item = "not";
first.next = oldfirst;
```

#### 表头删除结点

只需要将first指向first.next即可.但是一般来说你可能希望获取到删除元素的值,一旦改变了first的值,就再也无法访问曾经指向的结点,曾经的结点对象变成了一个孤儿,Java内存管理系统将回收他占用的内存.这个操作值含有一条赋值语句,运行时间和链表的时间和链表的长短无关

```java
first = first.next;
```

#### 表尾插入结点

要想在链表结尾加入新结点,首先需要找到最后一个结点.在链表中不要草率的决定维护一个额外的链接,因为每个修改链表的操作都需要添加检查是否要修改该变量的代码(只有一个结点的链表(首结点就是尾结点)和空链表(空连接),这些情况会让链表代码特别难以调试)

```java
Node oldLast = last;
last = new Node();
last.item = "not";
oldLast.next = last;
```

#### 其他位置的插入删除

对于指定结点插入和删除就比较不容易实现了.比如删除最后尾结点.last链接并不能帮助我们,因为这需要前一个结点,但是没有其他信息的前提下并不能快速的找到这个结点,唯一的方法就是遍历链表,这种方法效率并不是很理想.实现任意插入和删除的操作的标准解决方案是实现双向链表:每个结点都含有两个链接,分别指向不同的方向

```java
// 遍历列表的方法
for (int x = first; i != null; x = x.next){
    // 处理x.item
}
// 这可以是得迭代访问链表的所有元素而无需知道链表的实现细节
```

### **栈的实现:**

```java
// 下压堆栈(链表实现)
public class Stack<Item> implements Iterable<Item>{
    private Node first;  // 栈顶,最近添加的元素
    private int N;  // 元素数量
    private class Node{
        Item item;
        Node next;
    }
    public boolean isEmpty(){ return first == null;}  // 或者N == 0;
    public void push(){
        // 利用栈实现链表添加元素
        Node oldFirst = first;
        first = new Node();
        first.item = "添加的元素值";  // 
        first.next = oldfirst;  // 指向原来的第一个元素而自己成为第一个元素
        N++;
    }
    public Item pop(){
        // 利用栈实现删除元素
        Item item = first.item;
        first = first.next;
        N--;
        return item;
    }
    // Iterator算法实现:
    public Iterator<Item> iterator(){
        return new ListIterator;
    }
    private class ListIterator implements Iterator<Item>{
        // 构建一个类来实现迭代器
        // 构建实例变量,获取迭代器内的第一个方法
        private Node current = first;
        // 构建方法判断迭代器是否还有下一个元素
        public boolean hadNext(){
            return current != null;
        }
        // remove用来删除当前迭代器内的元素
        public remove(){ }
        // 实现迭代器的next方法
        public Item next(){
            Item item = current.item;
            current = current.next;
            return item;
            
        }
    }
}
```

### **队列的实现**

通过链表来实现队列也是比较方便的,实现先进先出的方法就是添加元素在尾结点添加,在首结点删除元素即可

```java
public class Queue<Item>{
    private Node first;  // 链表首元素
    private Node last; // 链表尾元素
    private int N;
    private class Node{
        // 定义了结点的嵌套类
        Item item;
        Node next;
    }
    // 判断队列是否为空
    public boolean isEmpty(){
        return first == null; // 或者N==0
    }
    // 获取队列大小
    public int size(){ return N;}
    // 往队列插入元素
    public void enqueue(Item item){
        Node oldlast = last;
        last = new Node();
        last.item = item;
        last.next = null;
        // 判断队列不为空才将上一个元素的next指向新增加的元素
        if ( !isEmpty() ){
	        oldlast.next = last;
    }else first = last;
        N++;
    }
    // 从队列中取出元素
    public Item dequeue(){
        Item item = first;
        first = first.next;
        if (isEmpty()) last= null;
        N--;
        return item;
    }
    // 获取迭代器
    public Iterator<Item> iterator(){
        return new QueueIterator();
    }
    // 实现迭代器的功能
    private class QueueIterator implements Iterator<Item>{
        // 构建一个类来实现迭代器
        // 构建实例变量,获取迭代器内的第一个方法
        private Node current = first;
        // 构建方法判断迭代器是否还有下一个元素
        public boolean hadNext(){
            return current != null;
        }
        // remove用来删除当前迭代器内的元素
        public remove(){ }
        // 实现迭代器的next方法
        public Item next(){
            Item item = current.item;
            current = current.next;
            return item;
            
        }
    }
}
```

### 背包的实现

使用链表数据结构实现背包只需要将Stack中的push()改名为add()并去掉pop()的实现即可

```java
public class Bag<Item> implements Iterable<Item>{
    private Node first;  // 链表首结点
    private class Node{
        // 定义结点的代码实现,可以是任意数据类型
        Item item;
        Node next;
    }
    public void add(Item item){
        // 和栈的push方法实现完全相同
        Node oldfirst = first;
        first = new Node();
        first.item = item;
        first.next = oldfirst;
    }
    // 获取迭代器
    public Iterator<Item> iterator(){
        return new BagIterator();
    }
    // 实现迭代器
    private class BagIterator implements Iterator<Item>{
        private Node current = first;
        public boolean hasNext(){
            return current != null;
        }
        // 迭代方法
        public Item next(){
            Item item = current.item;
            current = current.next;
            return item;
        }
    }
}
```

### 综述

<font color="red">结构化存储数据集时,链表是数组的一种重要替代方式. </font>虽然链表有不少优点,但是链表编程中也有不少问题,调试十分困难

有两种比较基础的对象集合的方式:

- 链表:链式存储
- 数组:顺序存储

后面的许多数据结构都是在基础的数据结构的基础上进行结归并扩展.

- 其中一种比较重要的扩展就是包含多个链接的数据结构。比如二叉树（有两个链接的结点组成）
- 另外一种重要的扩展是复合型的数据结构：使用背包存储栈，队列存储数组，如图（用数组的背包实现）用这种方式很容易定义任意复杂的数据结构，重点研究抽象数据类型就是试图控制这中复杂度

| 基础数据结构 |              优点              |              缺点              |
| :----------: | :----------------------------: | :----------------------------: |
|     数组     |  通过索引可以直接访问任意元素  | 在初始化时就需要知道元素的数量 |
|     链表     | 使用的空间大小和元素数量成正比 |    需要通过引用访问任意元素    |

设计抽象数据类型的步骤:

- 定义设计API：首先需要提前设计想好自己需要的一些API，随后再去写具体的API实现逻辑
- 根据特定的应用场景开发实际用例代码
- 描述一种数据结构，并在API对应的抽象数据类型的实现中根据它定义类的实例变量
- 描述算法（实现操作的方法），在这里去写具体的实现代码逻辑
- 分析算法的性能特点
