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