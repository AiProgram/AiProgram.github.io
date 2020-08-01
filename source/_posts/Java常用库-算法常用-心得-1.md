---
title: Java常用库(算法常用)心得-1
date: 2020-07-24 09:23:26
tags:
categories:
- Java
---

# Java常用库(算法常用)-心得1

**本文章主要总结最基本的Java库的使用，且在算法编程中经常用到**

<!-- more -->

## Collection、Map接口关系

+ 我们常说的Collection、Map的各种类实际上是实现了Collection、Map接口的各种实现类。
+ Collection和Map接口都位于 *java.util* 下，它们之间不是实现的关系。
+ 它们的实现类也常常位于 *java.util* 下，注意**不是位于** *java.util.Collection/Map* 下！
  + 由于多线程编程的特殊性，它们的实现类经常在 *java.util.concurrent* 下有着并发的实现类，形如`xxBlockingXX`、`xxConcurrentXX`。

## Java中的队列
+ Java中的队列最容易想到*java.util.Queue*，但是实际上这只是标准的Queue接口，是**不能直接使用**的。我们需要寻找的是实现了这个接口的类。同理的还有双端队列*java.util.Deque*接口，我们也只能用它的实现类，注意：`Deque`接口实现了`Queue`接口，所以`Deque`实现类全都可以当成`Queue`使用。
+ `Deque`作为单端队列使用时，常用方法有进队列(进队尾)：`offer(E)`、出队列(取队首并删除)：`poll()`、取队首`peekFirst()`对应取队尾`peekLast()`。而作为双端队列使用时，以上的方法均转为`XXFirst()`、`XXLast()`方法。
+ `Deque`还可以作为Stack使用，这时方法有入栈：`push(E)`，出栈`pop()`，取栈顶`peek()`。

1. 实现了`Deque`接口的类，在 *java.util* 下的有**ArrayDeque**和**LinkedList**。
2. 还有一个很重要的：优先队列，位于 *java.util* 包下，名为`PriorityQueue`，它实现了`Queue`接口。它的顺序按照排序指定，默认情况下取出元素是升序，我们可以利用 Comparator 接口或者 Comparable接口指定排序。
   1. 注意：PriorityQueue使用Iterator遍历是不保证顺序的，可以使用`toArray()`方法转换为数组再排序遍历。

## Java中的链表

### List接口

+ List接口提供了几种基础功能

```java
boolean add(String item)  //依次往后添加添加元素
boolean add(String item, int index) //在指定位置处添加元素
void remove(int position) //删除第几个元素（索引从0开始）
void remove(String item) //删除相同的元素
void removeAll() //删除所有元素
E set(int index, E e)//修改指定未知元素，返回原来值
```

+ 需要注意的是：

  1. `add`方法（不指定位置）默认是在末尾添加元素，这点可以由ArrayList特性记忆。
  2. `add(item,index)`方法是用插入元素取代index位置，原index及其后元素均后移一位。同样可以由ArrayList记忆。

+ 实现了List接口常见类有`ArrayList`、`LinkedList`、`Vector`、`Stack`。其中Vector和Stack使用了低效的同步策略，现在基本不用了被ArrayList和Deque替代。

+ List统统使用`java.util.Collections.sort`排序，结合`java.util.Comparator`实现自定义顺序。**注意**：Comparator需要实现的是`int compare()`方法而不是`compartTo()`方法。

+ List与数组的互相转换：

  1. List转换为数组：使用集合的方法`toArray(T[] array)`，例如：

  ```java
  String[] array = list.toArray(new String[list.size()]);
  ```

  这里使用了泛型，无需强制转换类型。如果使用无参`toArray()`则会出现强制转型错误。

  2. 数组转换为List：使用集合方法`Collections.addAll(arrayList, strArray)`，例如：

  ```java
  ArrayList< String> arrayList = new ArrayList<String>(strArray.length);
  Collections.addAll(arrayList, strArray);
  ```

  如果使用`asList()`则会出现获得的List无法增删的问题，因为该List是静态的。

+ List合并：
  1. 使用`List.addAll(Collection c)`方法，会把输入List全部添加到原List。
  2. 使用`List.retainAll(Collection c)`方法，会取交集，只保留两个List相同的元素。

### LinkedList

+ LinkedList由于其特殊的链表结构，推荐遍历方式是使用Iterator遍历，但是它的**ListIterator**功能更加强大。
  + `List.listIterator(int index)`可以获得指定位置的迭代器。
  + `ListIterator.hasNext()/next()`常用于向后遍历，向前则是`hasPrevious()/previous()`
  + ListIterator也有`add(E),remove(),E set(E e)`方法，建议使用。
  + 注意：ListIterator中，`remove()`的上个方法必须是`next/previous`，不能是`remove/add`。`add`不会影响下一次`next`但是会影响`previous`