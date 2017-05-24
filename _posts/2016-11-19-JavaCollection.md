---
layout: post
title:  Java集合
date:   2016-10-19 1:29:08 +0800
tag: 数据结构
---

## 基本概念
Java容器类类库的用途是“保存对象”，并将其划分为两个不同的概念：
1）Collection。一个独立元素的序列，这些元素都服从一条或多条规则。List必须按照插入的顺序保存元素，而Set不能有重复元素。Queue按照排队规则来确定对象产生的顺序（通常与他们被插入的顺序相同）。

2）Map。一组成对的“键值对”对象，允许你使用键来查找值。ArrayList允许你使用数字来查找值，因此在某种意义上讲，它将数组与对象关联在一起。映射表允许我们使用另一个对象来超找某个对象，它被称为“关联数组”，因为它将某些对象与另外一些对象关联在一起，或者被称为“字典”，因为你可以使用键对象来查找值对象，就像在字典中使用单词来定义一样。
###存储对象顺序的区别
Collection和Map的主要区别在于容器中的每个“槽”保存的元素个数。Collection在每个槽中只能保存一个元素。此类容器包括：

List,它以特定的顺序保存一组元素。Set，元素不能重复。Queue,只允许在容器的一“端”插入对象，并从另外一段移除对象。Map在每个槽内保存了两个对象，即键和与之相关联的值。

ArrayList和LinkList都是【按照被插入的顺序保存元素】。

HashSet无明显顺序添加元素（有最快的获取元素方式），TreeSet按照比较结果的升序保存对象，LinkedHashSet按照被添加的顺序保存对象。

HashMap与HashSet一样，HashMap也提供了最快的查找技术，也没有按照任何明显的顺序来保存其元素。TreeMap按照比较结果的升序保存键，而LinkedHashMap则按照插入顺序保存键，同时保留了HashMao的查询速度。

``` bash
Collection
├List
│├LinkedList
│├ArrayList
│└Vector
│　└Stack
└Set
Map
├Hashtable
├HashMap
└WeakHashMap
```
###1.1List
List接口在Collection的基础上添加了大量的方法，使得可以在List的中间插入和移除元素。

1）ArrayList:它长于随机访问元素，但是在List中间插入和移除元素时比较慢。因为它是一个数组队列，相当于动态数组。在数组中获取某个元素非常快，因为知道下标就可以获取。但是在中间插入数据或者移除数据的话，就比较慢。ArrayList中是非线程安全的，所以在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

2）LinkList在执行插入和移除时比ArrayList更高效，但在随机访问操作方面却要逊色一些。因为它是双向链表，可以当作堆栈、队列或双端队列进行操作。LinkList是非同步的。每个节点除含有元素外，还包含向前，向后的指针。

3）Vector 是矢量队列，它是JDK1.0版本添加的类。继承于AbstractList，实现了List（支持相关的添加、删除、修改、遍历等）, RandomAccess（随机访问功能）, Cloneable（能被克隆）这些接口。
Vector实际上是通过一个数组去保存数据的。当我们构造Vecotr时；若使用默认构造函数，则Vector的默认容量大小是10。
当Vector容量不足以容纳全部元素时，Vector的容量会增加。若容量增加系数 >0，则将容量的值增加“容量增加系数”；否则，将容量大小增加一倍。
Vector的克隆函数，即是将全部元素克隆到一个数组中。和ArrayList不同，Vector中的操作是线程安全的。
 
4）Stack 栈通常指“后进先出”（LIFO）。Stack继承于Vector(矢量队列)的，由于Vector是通过数组实现的，这就意味着，Stack也是通过数组实现的，而非链表。

###1.2 Set








