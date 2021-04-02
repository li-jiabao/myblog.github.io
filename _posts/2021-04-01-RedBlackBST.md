---
title: 红黑二叉树
author: lijiabao
date: 2021-04-02 23:30
categories: ["查找", "DataStruture and Alogrithm"]
tags: ["查找", "DataStruture and Alogrithm"]
---

介绍红黑二叉树的实现

## 红黑二叉树

使用标准的二叉查找树（完全由2-结点构成）和额外的信息（替换3-结点：由一种特殊链接连接两个2-结点组成的）来表示。

### 替换3-结点

红黑二叉树的链接分为两种类型:

- 红链接将两个2-结点连接起来构成一个3-结点
- 黑链接就是二叉树种的普通链接

![红链接表示3-结点](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E7%BA%A2%E9%93%BE%E6%8E%A5%E8%A1%A8%E7%A4%BA%E7%9A%843-%E7%BB%93%E7%82%B9.png)

红黑树中的3-结点由一条左斜的红色链接（2个2-结点的其中一个是另外一个的左子结点）相连的两个2-结点。这样表示，可以直接利用二叉树的查找算法而不需要修改。

### 红黑树等价定义和一一对应关系

红黑树的另外一种等价定义：含有红黑链接并且满足下面条件的二叉查找树：

- 所有的红链接均为左链接
- 没有任何一个结点同时和两个红链接相连
- 该树是完美黑色平衡的，任意空链接到根节点路径上的黑色链接数量相同

如果将红链接相连接的两个结点合并（红链接展平），得到的就是2-3树的3-结点，如果将2-3树的3-结点变成两个由红链接连接的3-结点，因此不会存在能够和两个红链接相连的结点。树必然是完美黑色平衡的：红黑树既是二叉查找树，也是2-3树。如果可以保证上述的一一对应关系来实现插入算法，就可以结合两种树的优点，实现二叉树的高效查找和2-3树的高效插入的结合的平衡树算法。

### 红黑树颜色表示

对于链接的颜色表示：使用一个名为color的布尔类型变量表示，如果指向它的链接是红色的，color为 true，否则为false，对于<font color='red'>空链接默认设定为黑链接</font>。

设置两个常量RED和BLACK来表示，并且添加一个isRed方法来判断结点和父结点之间链接的颜色。

```java
public class RedBlackBST<Key extends Comparable<Key>. Value>
{
    private static final boolean RED = true;
    private static final boolean BLACK = false;
    class Node
    {
        private Key key;
        private Value value;
        private Node left,right;
        private boolean color;
        private int N;
        // Node类的构造方法
        public Node(Key key, Value value, int N, boolean color)
        {
            this.key = key;
            this.value = value;
            this.N = N;
            this.color = color;
        }
    }
    // 判断结点和父结点之间链接的颜色
    public boolean isRed()
    {
        if (x == null) return BLACK;
        return x.color == RED;
    }
}
```

### 红黑树和2-3树一一对应的操作

为了保证红黑树和2-3树的一一对应，需要进行一些特殊的操作来保证：

- 旋转操作
- 颜色转换

#### 旋转

可能在实现某些操作的过程中会出现红色的右链接或者两条连续的红链接，但是在操作完成之前这些情况都会被小心的旋转修复。旋转操作会改变红链接的指向。

左旋转：一条红色的右链接需要被转换为左链接的操作。左旋转的操作通过将根节点转换为键值更大的结点，然后再按照大小顺序安排其他链接的链接结点和位置。

右旋转：和左旋转的操作十分类似，只需要将left改为right即可

![红黑树的旋转操作](C:%5CUsers%5Cjiabao%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20210402145026571.png)

不管左旋转还是右旋转，每次旋转操作都会返回一条链接，然后再将这个链接赋给父结点的链接。对于产生了两条连续的红链接的情况，还是会继续使用旋转来修正这种情况的

旋转操作是插入操作的一个简单的补充，旋转操作帮助我们保证2-3树和红黑树之间的一一对应关系。<font colot='red'>旋转操作可以保持红黑树的有序性和完美平衡性</font>，在红黑树进行旋转时奴需要担心有序性和完美平衡性。

还有两种红黑树的性质需要保证：红黑树中没有两个连续的红链接以及红黑树种没有红色的右链接

#### 颜色转换

需要定义一个颜色转换的方法：flipColors，这个操作需要可以做到红变黑，黑变红的操作。

这个颜色转换的操作和局部旋转变换一样，并不会改变红黑树有序性和黑色平衡性

![红黑树颜色转换](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E7%BA%A2%E9%BB%91%E6%A0%91%E7%9A%84%E9%A2%9C%E8%89%B2%E8%BD%AC%E6%8D%A2.png)

#### 2-结点插入新键

对于需要插入键的位置处是一个2-结点的情况，当你插入键时：

- 新键小于根节点，直接新键一个红色结点并放在根节点的左链接
- 新键大于根节点，位于右侧，插入红色结点之后需要进行左旋转

![2-结点的新键插入](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/2-%E7%BB%93%E7%82%B9%E7%9A%84%E6%96%B0%E9%94%AE%E6%8F%92%E5%85%A5.png)

#### 3-结点（双键树）插入新键

对于3-结点的键插入，又有三种子情况：

- 插入的键位于右子结点，这时候插入红色结点之后，变成两个红链接，而且还是右链接，将所有的红链接全部变成黑链接即可满足有序性和平衡型要求（不需要进行旋转操作，直接红黑链接转换即可）
- 插入的键位于左子结点：用红链接将新键链接，这时候出现两个连续的红链接，然后进行右旋转，变为红色右链接，此时又回到上面的第一种情况，转换为黑链接即可。（进行一次旋转操作和两次红黑链接转换）
- 插入的键位于中间子结点：用红链接连接，这是红色右连接，首先左旋转将其变为上面的第二种情况，然后在右旋转，最后将两个红链接转换为黑链接。（需要进行两次旋转操作和两次红黑链接转换）

![3-结点插入新键](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/3-%E7%BB%93%E7%82%B9%E6%8F%92%E5%85%A5%E6%96%B0%E9%94%AE.png)

#### 红链接的向上传递

在2-3树中，插入新键时碰见了3-结点，需要首先将3-结点变成4-结点，然后对4-结点进行分解，之后再将4-结点的中键传递给父结点，直至碰见2-结点或者到达根结点。

而在红黑树中，为了保证和2-3树的一一对应关系，也需要为红黑树进行类似的操作，在4-结点的分解以及两个红链接的颜色转换之后，需要将父结点的颜色转换为红链接，然后将这个红链接向上传递，直到遇见一个2-结点插入或者到达根节点进行分解（到达根节点之后，黑链接树的高度会增加1）

对于插入时的有序性和平衡性以及和2-3树的一一对应关系，我们需要进行的基本操作就是下面几种：

- 如果右链接红色而左链接是黑色，进行左旋转
- 如果左链接是红色且左子结点也是红色，进行右旋转
- 如果左右结点都是红色，进行颜色转换并将根节点转换为红色向上传递

#### 根节点颜色总是黑色

因为根节点并没有父结点，因此根节点的颜色并不会是红链接，即使变换之后根结点变成红链接，也需要进行颜色转换将根结点转换为黑色。

当红链接传递到根结点时，如果根结点是2-结点，传递上来的红链接会将根结点所在的子树变成3-结点。此时并不会增加黑链接数的高度。如果根节点为3-结点，红链接传递上来之后，需要进行4-结点分解，此时分解完之后进行颜色转换（需要向上传递红链接）颜色转换后的根节点变成红色，但是由于根结点不能是黑链接，需要对根结点颜色从红转换为黑色（因此，当根结点从红色转换为黑色时，树的高度需要增加1）

### 红黑树的实现

保证树的平衡性所需的操作：由下向上在每个经过的结点中进行

```java
public class RedBlackBST<Key extends Comparable<Key>. Value>
{
    private Node root;
    private static final boolean RED = true;
    private static final boolean BLACK = false;
    class Node
    {
        private Key key;
        private Value value;
        private Node left,right;
        private boolean color;
        private int N;
        // Node类的构造方法
        public Node(Key key, Value value, int N, boolean color)
        {
            this.key = key;
            this.value = value;
            this.N = N;
            this.color = color;
        }
    }
    // 判断结点和父结点之间链接的颜色
    public boolean isRed(Node node)
    {
        if (node == null) return BLACK;
        return node.color == RED;
    }
    // 查找操作
    public Value get(Key key)
    {
        return get(root, key);
    }
    // 实现查找操作
    private Value get(Node node, Key key)
    {
        if (node==null) return null;
        int cmp = node.key.compareTo(key);
        if (cmp > 0) return get(node.left, key);
        else if (cmp < 0) return get(node.right, key);
        else return node.value;
    }
    // 插入操作
    public void put(Key key, Value value)
    {
        root = put(root,key,value);
        root.color = BLACK；
    }
    // 插入操作的辅助实现
    private Node put(Node node, Key key, Value value)
    {
        if (node == null) return new Node(key, value, 1, RED);
        int cmp = node.key.compareTo(key);
        if (cmp > 0) node.left = put(node.left, key, value);
        else if (cmp < 0) node.right = put(node.right, key, value);
        else node.value = value;
        if (isRed(node.right) && isRed(node.left)) noed = rotateLeft(node);
        if (!isRed(node.left) && isRed(node.left.left)) rotateRight(node);
        if (isRed(node.left) && isRed(node.right)) flipColor(node);
        node.N = size(node.left) + size(node.right) + 1;
        return node;
    }
    // 左旋转操作算法
    private Node rotateLeft(Node node)
    {
        Node h = node.right;
        node = node.right;
        node.left = h;
        h.color = node.color;
        node.color = RED;
        h.N = node.N;
        node.N = size(node.left) + size(node.right) + 1;
        return node;
    }
    // 右旋转操作算法
    private Node rotateRight(Node node)
    {
        Node h = node.left;
        node.left = h.right;
        h.right = node;
        h.color = node.color;
        node.color = RED;
        h.N = node.N;
        node.N = size(node.left) + size(node.right) + 1;
        return node;
    }
    // 颜色转换
    private void flipColor(Node node)
    {
        node.left.color = BLACK;
        node.right.color = BLACK;
        node.color = RED:
    }
}
```

![红黑树构造路径](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E7%BA%A2%E9%BB%91%E6%A0%91%E7%9A%84%E6%9E%84%E9%80%A0%E8%B7%AF%E5%BE%84.png)



### 红黑树的删除实现

对于删除操作的实现，可以和插入操作实现一样，定义一系列的局部变换来删除一个结点的同时保持树的完美平衡性，但是这个过程比插入一个结点更加的复杂，因为不仅要在构造临时4-结点时沿着查找路径向下进行变换，还要再分解遗留的4-结点时沿着查找路径向上进行变换（和插入操作类似）

#### 自顶向下的2-3-4树

要使用红黑树实现这个算法，需要：

- 将四个结点表示为由3个2-结点组成的一棵平衡的子树，根结点和两个子结点都是用红链接相连
- 向下的过程中分解所有4-结点并进行颜色转换
- 和插入操作一样，在向上的过程中使用旋转将4-结点配平

对于该算法的实现，只需要将红黑树算法中的put方法一行代码移动就能实现插入操作：将flipColor移动到递归调用之前（null测试和比较操作之间）多个进程可以同时访问一棵树中的应用这个算法优于2-3树，因为总是操作当前结点的一个或者两个链接。

![自顶向下2-3-4树插入操作变换](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E8%87%AA%E9%A1%B6%E5%90%91%E4%B8%8B%E7%9A%842-3-4%E6%A0%91%E6%8F%92%E5%85%A5%E7%AE%97%E6%B3%95%E5%8F%98%E6%8D%A2.png)

#### 删除最小键

删除树底部的3-结点键很简单，但是2-结点不是，删除其中一个键之后，就会破坏树的完美平衡性，因此：为了保证不删除2-结点，沿着左链接向下进行变换确保当前结点不是2-结点（可以3-结点或者临时的4-结点）

根结点可能有两种情况：

- 根结点为2-结点，且他的两个子结点都是2-结点，这时候可以将三个节点合并为一个4-结点
- 不然需要保证根结点的左子结点不是2-结点，可以的话从左侧兄弟节点借一个键

沿着左链接向下的过程中，确保下列情况成立；

- 当前结点左子结点不是2-结点，完成
- 如果当前结点的左子结点是2-结点但是兄弟节点不是2-结点，从兄弟结点借一个键到左子结点，完成
- 当前结点的左子节点和兄弟节点都是2-结点，将左子结点、父结点的最小键和左子结点最近兄弟结点变成4-结点，使得父结点由3-结点变为2-结点或者4-结点变成3-结点

![2-3-4树删除最小键的操作变换](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/2-3-4%E6%A0%91%E5%88%A0%E9%99%A4%E6%9C%80%E5%B0%8F%E9%94%AE%E6%93%8D%E4%BD%9C%E5%8F%98%E6%8D%A2.png)

#### 删除操作

在查找路径上进行和删除最小见相同的变换同样可以保证查找过程中任意当前结点均不是2-结点。如果被查找的键在树的底部，可以直接删除，如果不在，需要将它和它的后继结点交换，和二叉树一样。因为当前结点必然不是2-结点，问题就转换为在一棵跟结点不是2-结点的子树中删除最小的键，我们可在这颗子树中使用上面提到的删除最小值的算法，删除之后需要向上回溯并分解余下的4-结点。

### 红黑树的性质

**<font color='red'>所有基于红黑树的符号表的实现都可以保证操作的运行级别是对数级别的（除范围查找外，所需的额外时间和返回的键的数量成正比）</font>**

#### 性能分析

无论如何插入数据，红黑树都几乎是完美平衡的，因为红黑树和2-3树是一一对应的

> 一棵大小为N的红黑树的高度不会超过2lgN
>
> <font color='red'>红黑树的最坏情况：</font>当红黑树对应的2-3树种构成最左边的路径结点全部都是替换3-结点，其余均为2-结点。因此最左边的路径长度是只包含2-结点的路径长度（lgN）两倍

> 在一棵大小为N的红黑树中，根结点到任意结点的平均路径长度为1.00lgN

#### 有序符号表API分析

红黑树最好的一个点就是它的实现中最复杂的代码只有put方法和删除方法。二叉查找树中查找最大值和最小值、select、floor、ceiling和范围查找方法不做任何变动可以继续使用在红黑树上，因为红黑树因为是一种二叉树，而且这些操作并不涉及结点颜色的改变。上面算法的简单实现中代码实现的方法和这些方法构成了红黑树有序符号表的API。<font color='red'>这些方法最多需要的时间都和树高成正比</font>

> 红黑树中，查找删除、查找最大和最小键、floor、ceiling、rank、select、delMax、delMin以及范围查找range，这些操作在最坏情况下所需的时间是对数级别

![各种符号表实现的性能总结](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E5%90%84%E7%A7%8D%E7%AC%A6%E5%8F%B7%E8%A1%A8%E5%AE%9E%E7%8E%B0%E7%9A%84%E6%80%A7%E8%83%BD%E6%80%BB%E7%BB%93.png)