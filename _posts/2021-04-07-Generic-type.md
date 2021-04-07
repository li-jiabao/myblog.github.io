---
title: Java泛型
author: lijiabao
date: 2021-04-07 19:30
categories: ["Java学习笔记"]
tags: ["Java学习笔记"]
---

介绍Java泛型的优点，泛型的定义方法，Java泛型的变量限定，Java泛型的类型擦除，Java泛型通配符，Java泛型的局限性，最后介绍了泛型的不可靠类型已经不可靠类型可能造成堆污染的情况。

## Java泛型

### 泛型类型变量和原始类型

> **泛型类型变量**：是泛型类或者接口中的参数化后的类型

> **参数化类型**：就是赋给了参数的泛型类或者接口，如`Comparable<Double>`、`List<String>`

> **原始类型**（Raw type）：是没有类型参数的泛型类或者接口的称呼`List`是`List<string>`的原始类型

**泛型类型变量的命名惯例：**

- E - 元素，广泛用于Java集合框架中
- K - Key 键
- N - number数值类型
- T - 普通类型，第一个类型变量
- V - Value 值
- S、U、V - 第二、三、四个类型变量

### 使用泛型优点

泛型程序设计可以让编写的代码对多种不同类型的对象重用，这样就不必要为很多具有相同操作的类型对象分别创建各自的类。这样做有以下的优点：

- 代码复用：一套代码可以用于不同的对象类型

- 代码更加清晰易懂

- 降低了耦合性

- 减少类型转换，对于未使用泛型的代码，往往可能需要强制类型转换，但是泛型代码不需要

  ```java
  // 非泛型代码的使用
  List list = new ArrayList();
  list.add("hello");
  String s = (String) list.get(0);
  // 泛型代码的使用
  List<String> list = new ArrayList<>();
  list.add("hello");
  String s = list.get(0);
  ```

- 更高的安全性：Java编译器对泛型代码有着强类型检查，一旦代码违反了类型安全们就会报错。编译型错误远比运行时错误更好发现和解决

### 泛型类

泛型类就是有一个或者多个类型变量的类，定义了泛型类，就不用再为类的变量类型等细节而烦扰，只需要关注代码的泛型设计即可。

```java
public class ClassName<类型变量>
{
    public ClassName(){}
}
// 示例
// 其中的Key和Value就是Java泛型中的类型变量
// 类型变量用于在类中指定方法的返回类型以及字段和局部变量的类型
public class Transaction<Key, Value>
{
    private Key key;
    private Value value;
    public Transaction()
    {
        this.key = new Key();
        this.value = new Value();
    }
}
```

其实泛型类就相当于普通类的 工厂，泛型类在实际使用的时候可以指定特定的类型变量来实例化泛型

```java
// 第一种实例化泛型类的方法
Transaction<String, Integer> transaction = new Transaction<String, Integer>();
// 第二种泛型类实例化，使用菱形语法，可以忽略构造器中类型变量的指定，减少代码量
Transaction<String, Integer> transaction = new Transaction<>();
// 第三种泛型实例化，使用var关键字，可以忽略变量类型的指定
var transaction = new Transaction<String, Integer>();
// 后面两种实例化都是利用编译器通过现有的代码信息来推测的类型变量，比较方便，可以减少代码量
```

### 泛型方法

泛型方法定义的格式：

`[修饰符] <类型变量>  [返回值]  [方法名](参数类型，可以使用类型变量){}`

```java
public class Util
{
    public static <K, V> boolean funcName(Transaction<K, V> a1, Transaction<K, V> a2)
    {
        return a1.key.compareTo(a2.key) > 0;
    }
    // 上述定义的方法就是一个泛型方法示例
}
```

泛型方法的使用：

```java
public class GenericMethodsTest
{
    public static void main(Stirng[] args)
    {
        var a1 = new Transaction<String, Integer();
        var a2 = new Transaction<String, Integer();
        boolean test = Util.<String, Integer>funcName(a1, a2);
        // 也可以不显示的指定，可以让编译来推测应该使用什么类型
        boolean test = Util.funcName(a1, a2);
    }
}
```

上面的泛型类和泛型方法都提到了编译器的类型推测：泛型的类型推测是Java编译器进行的操作，编译器根据上下文信息，也就是Java的类型声明（应用于泛型类的推测）和类型参数（应用于泛型方法的推测）来推测应该使用哪一种类型来编译，编译器会找到最合适的类型来进行编译。

### 类型变量的限定

当你需要对类型变量进行约束的时候，可以使用类型变量的限定。如`public class ClassName<T extends Number>`这个声明指定了Java的类型变量的上界是Number类，也就是说T只能是Number类和它的子类，在这里这个extends和继承的类似的，可以同时用于类和接口。

```java
// 单类型限定的使用，类和方法的用法
public class Transaction<T extends Number>
{
    public static <T extends Number> boolean comapre(T t1, T t2)
    {
        return t1 > t2;
    }
    // 可以指定参数化类型的限定继承,如此定义之后就可以使用compareTo方法了
    public static <T extends Comparable<T>> boolean comapre(T t1, T t2)
    {
        return t1.compareTo(t2) == 0;
    }
}
```

类型变量限定还支持多个变量限定：`<T extends Class1 & Class2 & Interface1 & Interface2>`

### Java泛型的继承和子类

泛型当中的继承关系和正常的继承关系优点不一样，像下图中，尽管Integer和Number是继承关系，但是以这二者为类型参数的泛型并不是继承关系。

![泛型继承关系](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E6%B3%9B%E5%9E%8B%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB.png)

泛型类是可以继承泛型类或者泛型接口的，这些继承的规则和正常类的继承规则一样。比如：`ArrayList<T>`实现了`List<T>`接口，因此，可以将使用类似的继承和多态`List<Number> ints = new ArrayList<Number>`，但是也和前面说的一样，`ArrayList<Integer>`并不是`ArrayList<Number>`

![](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E6%B3%9B%E5%9E%8B%E7%B1%BB%E7%AD%89%E7%BA%A7%E5%88%B6%E5%BA%A6.png)

### Java泛型通配符

泛型通配符是`?`，代表的是未知类型

通配符使用：

- 作为参数、字段或者局部变量的类型
- 返回类型（但是明确的类型是更好的编程习惯）

<font color='red'>通配符不能用于泛型方法调用时的类型参数，也不可以用在实例创建，作为超类也是不允许的</font>

#### 上边界通配符

`<? extends Number>`，当你想要实现支持某个类或者他们子类的类型时，可以使用上边界符，如：`List<? extends Number>`，number就是上边界

#### 无边界通配符

单独使用`?`作为无边界通配符，可以指代任意类型，使用场景：

- 当你需要写一个Object类提供的功能实现的方法时
- 泛型类中的方法并不依赖于类型参数

#### 下边界通配符

`<? super Integer>`当你想要实现支持某个类或者该类的父类类型时，可使用下边界符

#### 通配符中继承的等级制度

在泛型类继承的等级制度中，我们了解到泛型类的等级制度和标准的等级制度略有差异，如`Box<Integer>`并不是`Box<Number>`的子类。但是如果利用通配符，可以实现类似的等级制度。

![](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/%E6%B3%9B%E5%9E%8B%E9%80%9A%E9%85%8D%E7%AC%A6%E4%B8%AD%E7%9A%84%E7%AD%89%E7%BA%A7%E5%88%B6%E5%BA%A6.png)

#### 通配符捕获和辅助方法

通配符捕获：当使用通配符的的情况下进行表达式的评估，编译器会从代码中的信息推测类型，这个场景就是通配符捕获

```java
import java.io.util.List;
public class WildcardError
{
    void foo(List<?> i)
    {
        i.set(0., i.get(0));
    }
}
// 这段代码会报错
```

上述代码报错的原因是：当编译器处理i参数时认为这个是Object类型，当调用set方法，编译器并不能确保对象类型插入到了列表，就报错.

上述情况的解决方案，提供一个辅助方法

```java
public class WildcardFixed
{
    void foo(List<?> i)
    {
        fooHelper(i);
    }
   // 创建辅助方法以保证编译器通过类型引用完成通配符捕获
    private <T> void fooHelper(List<T> l)
    {
        l.set(0, l.get(0));
    }
}
```

#### 通配符使用原则

In变量：提供数据给代码的变量，如`copy(src, des)`方法中的src，就是in变量，提供数据的变量

Out变量：持有数据给其他地方使用的变量，如`copy(src, des)`，这个des是持有数据的变量，因此是out变量

通配符使用指导原则：

- in变量定义使用上边界通配符，使用extends关键字定义
- out变量使用下边界通配符，使用super关键字
- 当in变量可以使用Object方法定义的方法访问时，使用无边界通配符
- 当这个变量既要作为in也要作为out变量访问，就不要使用通配符了

### 泛型类型擦除

类型擦除是编译器提供的，为了正常的兼容泛型代码：

- 替代所有的类型参数为边界类型参数或者没有限定类型时变成Object，擦除之后的字节码只有普通的类接口和方法，不再存在泛型
- 必要时为了保护类型安全会使用强制类型转换
- 在泛型方法中为了维护多态，使用合成桥方法

泛型擦除确保了不同参数化类型不会产生新的类，因此泛型不会产生运行时开销

#### 泛型类的擦除

当你并没有限定类型时的类型擦除：

```java
// 无限定泛型代码
public class Node<T>
{
    private T data;
    private Node<T> next;
    public Node(T data, Node<T> next)
    {this.date = data;this.next=next;}
    public T getData(){}
}
// 擦除后代码
public class Node
{
    private Object data;
    private Node next;
    public Node(Object data, Node next){...}
    public Object getData(){}
}
```

当使用了类型限定时的类型擦除：

```java
// 泛型代码
public class Node<T extends Comparable<T>>
{
    private T data;
    private Node<T> next;
    public Node(T data, Node<T>){}
    public Object getData(){return data;}
}
// 擦除后的代码
public class Node
{
    private Comparable data;
    private Node next;
    public Node(Comparable data, Node next){}
    public Comparable getData(){return data;}
}
```

#### 泛型方法的类型擦除

对于泛型方法，编译器同样也会进行擦除：

```java
// 擦除前
public static <T> int count(T[] anArray, T elem)
{
    int cnt = 0;
    for (T e : anArray)
        if (e.equals(elem))
            ++cnt;
    return cnt;
}
// 擦除后
public static int count(Oject[] anArray, Object elem)
{
    int cnt = 0;
    for (Object e : anArray)
        if (e.equals(elem))
            ++cnt;
    return cnt; 
}
// 限定类型的泛型方法擦除
public static <T extends Shape> void draw(T shape){}
// 擦除后
public static void draw(Shape shape){}
```

为了保持Java继承中的多态特性，Java的编译器会在类型擦除时生成合成桥方法。因为在类型擦除之后，方法签名并不匹配，因为并没有覆盖重写父类中的Object的方法，因此并不满足多态重写要求，为了保证正常，编译器就添加了一个Object的合成桥方法，来解决继承问题并保持多态，使得子类可以如期待一样工作。

继承方法覆盖时，可以指定一个更严格的返回类型，这是合法的，这种方法被称之为<font color='red'>有协变的返回类型</font>

```java
class Node<T>
{
    public T data;
    public Node(T data) { this.data = data; }
    public void setData(T data){ this.data = data;}
}
public Class MyNode extends Node<Integer>
{
    public MyNode(Integer data) { super(data); }
    public void setData(Integer data) { super.setData(data);}
}
// 类型擦除后
class Node
{
    public Object data;
    public Node(Object data) { this.data = data; }
    public void setData(Object data) { this.data = data; }
}
public class MyNode
{
    public MyNode(Integer data) { super(data); }
    public void setData(Integer data) { super.setData(data); }
    // 合成桥方法
    public void setData(Object data) { setData((Integer) data);}
}
```

#### 泛型表达式的强制类型转换

对于泛型方法调用，如果擦除了返回类型，编译器会自动插入强制类型转换

对于泛型字段的访问也要插入强制类型转换，这些都是编译器自动进行的。当然也可以自己手动插入。

### 泛型的不足和局限性

泛型主要有以下的一些限制和局限：

- 不能使用原始数据类型实例化泛型类
  `Node<int> node = new Node(1);`这是不可以的，编译错误

- 不能创建类型参数的实例
  直接实例化不可行，编译错误，但是可以利用反射来获取实例

  ```java
  public static <E> void append(List<E> list)
  {
      E elem = new E();  // 编译时错误
  }
  // 合法的参数化类型的实例获取
  public static <E> void append(List<E> list, Class<E> cls) throws Exception
  {
      E elem = cls.newInstance();  // 这种获取实例的方法是可行的
  }
  list<String> ls = new ArrayList<>();
  append(ls, String.class);  // 调用，传入String的反射类
  ```

- 不能声明类型是类型参数的静态变量

  如果允许这样的静态变量，但又因为静态变量是类中对象所共有，当有多个不同类型参数的对象共存时，会造成困惑，不能确定最终该静态变量的类型，因此，<font color='red'>不允许使用类型参数创建静态字段</font>

- 不能用参数化类型进行类型转换和instanceof判断
  instanceof只能用于reifiable类型判断，如下第二种。

  ```java
  public static <E> void (List<E> list){
      list instanceof ArrayList<Integer>;  // 出现编译错误
  }
  public static <E> void (List<？> list){
      list instanceof ArrayList<？>;  // 可以编译成功
  }
  // 继承关系类多态特性
  List<Integer> li = new ArrayList<>();
  List<Number> ln = li;  // 编译错误，并不具备继承关系
  List<String> li = new ArrayList<>();
  ArrayList<String> ln = li; // 可编译，二者存在继承关系
  ```

- 不能创建参数化类型的数组

- 不能创建、捕获或者抛出参数化类型的对象

- 对于参数化类型（每次重载擦除之后得到同一个原始类型）的情况，不能重载该方法

  ```java
  public void print(Set<String> strSet) {}
  public void print(Set<Integer> intSet) {} 
  // 上述两个方法在类型擦除后得到的是同一个原始类型Set，并不满足重载要求，编译错误
  ```

### 非可靠类型和可靠类型

可靠类型（reifiable类型）：类型信息在运行时完全可以获取的类型称为可靠类型

非可靠类型（Non-reifiable）：指的是在编译时由于类型擦除信息已经倍移除的类型——没有定义为无边界通配符的泛型方法。非可靠类型在运行时并不拥有它所有信息。不可靠类型的例子：`List<String>`和`list<Number>`Java虚拟机并不能区分。

不可靠类型不可用的场合：

- instanceof判断符不能用于非可靠类型
- 非可靠类型也不可以用于数组元素

#### 堆污染

当一个参数化类型变量指向的不是这个参数化类型时，会发生堆污染。如果程序执行某些操作在编译时出现了非检查型异常，堆污染也会发生。如果产生了非检查型警告（可以是编译时类型检查规则之下进行编译时，也可以是运行时），无法验证涉及参数化类型操作（强制类型转换或方法调用）的正确性。例如：当混合了原始类型和参数化类型的时候或者执行非检查类型转换的时候。

#### 可变参数问题

非可靠类型用在可变参数中可能会出现污染，包含可变参数输入可能会造成堆污染

```java
public static <T> void addAll(Collection<T> coll, T... ts)
{
    for (T t : ts) coll.add(t);
}
Collection<Box<String>> table = ...;
Box<String> box1 = ...;
Box<String> box2 = ...;
addAll(table, box1, box2);
// 这里会出现Varargs警告信息
```

堆污染的例子：

```java
public class HeapPollutionExample {

  public static void main(String[] args) {

    List<String> stringListA = new ArrayList<String>();
    List<String> stringListB = new ArrayList<String>();

    ArrayBuilder.addToList(stringListA, "Seven", "Eight", "Nine");
    ArrayBuilder.addToList(stringListB, "Ten", "Eleven", "Twelve");
    List<List<String>> listOfStringLists =
      new ArrayList<List<String>>();
    ArrayBuilder.addToList(listOfStringLists,
      stringListA, stringListB);

    ArrayBuilder.faultyMethod(Arrays.asList("Hello!"), Arrays.asList("World!"));
  }
}
```

#### 可变参数警告避免

对于上述的Varargs的警告信息，怎么避免：

- `@SuppressWarnings({"unchecked", "varargs"})`加入这个注解到方法声明前
- `@SafeVarargs`加入这行注解到方法声明