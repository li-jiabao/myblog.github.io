---
title: 二叉查找树
author: lijiabao
date: 2021-04-01 15:30
categories: ["查找", "DataStruture and Alogrithm"]
tags: ["查找", "DataStruture and Alogrithm"]
---

介绍二叉查找树的实现方法，以及二叉查找树的效率。二叉查找树是一种结合了有序数组的查找效率和链表插入效率的数据结构。

## 二叉查找树

二叉查找树的具体实现：就是使用链表的数据结构表示，但是链表中的每个结点都有两个链接（一个指向比结点小的左子树，一个指向比自己大的右子树）

数据结构的主要实现和链表一样，都是通过结点类来实现的，每个结点类中有两个链接（分别指向自己的左子结点和右子结点），结点的链接可以指向null或者其他结点。二叉树中，每个结点只能有一个父结点指向自己（根结点是没有父结点）

尽管每个链接都是指向一个结点，但是这个结点可以看成是另外一个二叉树（这颗二叉树的根结点就是指向的结点）

二叉树定义：可以是一个空链接，也可以是一个有左右两个链接的结点（子二叉树）

二叉树中的每个结点除了包含两个链接之外，还包含键（可以是多个键，包含多个键可以实现多键分别排序）和值，键之间有顺序之分可以支持高效的查找

> 二叉查找树是一颗二叉树，其中的每个结点含有Comparable的键（和键关联的值），每个结点的键的值大于左子树的任意结点键，而小于右子树的任意结点键

### 基本实现

```java
public class BST<Key extends Comparable<Key>, Value>
{
    private Node root;
    // 定义二叉树中的结点类，用来创建两个链接和键值对
    private class Node
    {
        private Key key;  // 结点对应的键
        private Value value;  // 结点对应的值
        private Node left,right;  // 结点左右子链接，分别连接左右子树
        private int N;  // 可有可无，代表结点下的子结点数目
        // Node类的构造方法
        public Node(Key key, Value value, int N){
            this.key=key; this.value=value;this.N = N
        }
    }
    // 二叉树大小
    public int size(){
        return size(root);
    }
    // 查看结点下的结点数量
    public int size(Node node)
    {
        if (x==null) return 0;
        else return x.N;
    }
    // 二叉树查找，从根节点往下查找
    public Value get(Key key)
    {
        return get(root, key);
    }
    /**
    * 二叉树查找算法实现，可选取开始查找的根结点
    * 这个算法实现属于辅助性代码，不需要给外界访问的，因此需要设置为private私有方法
    * 因为需要比对结点的值，所以增加一个重载方法进行更方便的比较操作和判断
    * 增加结点参数之后，可以更方便的切换到到左子树或者右子树再度进行递归
    */
    private Value get(Node node, Key key)
    {
        // 自己写的一个实现，考虑的不是很周到，代码不简洁
        // 代码的简洁化，属于重构，基础学好了，框架中间件了解了，可以尝试看些重构相关的书籍
        int comp = node.key.compareTo(key)
        if (comp<0 && node.right!=null) return get(node.right, key);
        else if (comp>0 && node.left!=null) return get(node.left, key);
        else if (comp==0) return node.value;
        else return null;
        // 更简洁的实现
        // if (x==null) return null;
        // int cmp = node.key.compareTo(key);
        // if (cmp>0) return get(node.left,key);
        // else if (cmp<0) return get(node.right,key);
        // else return node.value;
    }
    // 二叉树插入操作
    public void put(Key key, Value,value)
    {
        put(root, key, value)
    }
    // 二叉树插入算法实现
    // 类似上面查找的算法实现逻辑，都是利用一个带结点参数的辅助方法提供帮助
    private void put(Node node, Key key, Value value)
    {
        // 首先查找结点是否存在，存在就更新结点的值
        // 结点不存在就新增一个结点并插入到树中
        if (node == null) return new Node(key, value, 1); // 结点不存在，将结点新增到树中
        int cmp = node.key.compareTo(key);
        if (cmp>0) node.left=put(node.left, key, value);
        else if (cmp<0) node.left=put(node.right, key, value);
        else node.value=value;
        node.N = size(node.left) + size(node.right) + 1;
        return node;  // 这个return 语句是返回存在相同结点时更新操作后结点的
    }
    // 其他有序性相关操作
    // max()、min()、floor()、ceiling()
    // select()、rank()
    // delete()、deleteMin()、deleteMax()
    // 获取键的迭代器keys()
}
```

### 二叉树分析

使用二叉树的算法的运行时间取决于树的形状，树的形状则又取决于键对应结点的插入顺序。

二叉树的最好情况就是树的结构是平衡的，每一条空链接和根节点之间的距离都是lgN，最坏情况就是搜索路径上有N个结点。但是一般情况下，树的结构更接近平衡二叉树。

<font color='red'>二叉查找树的构建模型和快速排序的构建模型类似</font>，二叉树的根节点（左子树小于根节点，右子树大于根节点）和快速排序的第一个切分元素（左边小于切分元素，右边大于切分元素）是一样的

> 在N个随机构建的二叉查找树中，查找命中平均所需比较次数为：2lgN
>
> 插入操作和查找未命中的平均所需要的比较次数为：2lgN
>
> 在二叉树中，所有操作在最坏情况下所需的时间都和树的高度成正比。

![简单的符号表实现的成本总结](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E7%AE%80%E5%8D%95%E7%AC%A6%E5%8F%B7%E8%A1%A8%E5%AE%9E%E7%8E%B0%E7%9A%84%E6%88%90%E6%9C%AC%E6%80%BB%E7%BB%93.png)