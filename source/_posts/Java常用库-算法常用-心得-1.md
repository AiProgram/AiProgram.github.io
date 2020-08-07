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

## Java中的String

### CharSequence

+ CharSequence是一个接口，String、StringBuilder、StringBuffer都实现了这个接口。
+ CharAt()、length()、toString()、toCharSequence()都是实现了这个接口的方法。

### String的内存模型及实现

+ String底层使用字符数组实现，字符数组使用final修饰说明它的引用不能修改，同时这个数组定义为String类的private变量，向外没有提供直接修改的方法。本身String提供的连接等函数都是新建了一个String返回的。
+ 综上所述我们说String是定义后就不变的，我们最多只能修改指向String类对象的地址。本身String对象是不会变的。**但是**：实际上，在违反封装性的前提下我们可以使用反射修改String内char数组，只不过这不属于正规方法。
+ **在JDK 1.8以后**，String常量池移动到了堆中。如果是直接使用`String a="hello"`定义的字符串，是放在常量池中的，如果是用`new`出来则放在常量池外部的堆中。但是new出来的字符串调用`intern()`获得的是常量池对应的引用。
+ **注意**：对应final修饰的String对象，编译器会依据不变的特性猜测，从而更大范围的利用常量池。例如

```java
String str1="a";
final String str2="a";
String str3=str1+"b";
String str4=str2+"b";
String str5="ab";
str3==str5;// false
str4==str5;// true
String str6=new String(str1);
str6==str1;
```

### String常用方法

+ Java中类似sprintf的字符串格式化方法：`String.format();`，用例见：

```java
String fs;
fs = String.format("浮点型变量的值为 " +
                   "%f, 整型变量的值为 " +
                   " %d, 字符串变量的值为 " +
                   " %s", floatVar, intVar, stringVar);
```

+ `charAt(int index)`返回指定位置字符。
+ `int compareTo(String anotherString)`:字符对比。
+ `String substring(int beginIndex, int endIndex)`:返回字串（$[begin,end)$）。
+ 获得对应的charArray:`char[] toCharArray()`。
+ 用charArray构造String：`String String.valueOf(char[] data, int offset, int count)`，注意：这里用起点和长度定义。
+ `String.strip()`：去除前后部的空格字符，这里考虑到了Unicode所以空格字符不仅仅是ASCII码中的空格和控制字符。
+ `public String[] split(String regex)`:用于切分字符串，返回的是字符串数组。
  + 注意：输入的是正则表达式，这意味着'\\'，'.'等正则式保留符号均需要转义才能正确工作。
  + 同时，用于分割的字符不会出现在结果中，例如 "boo:and:foo" 被“:”分割得到 { "boo", "and", "foo" }，被"o"分割得到 { "b", "", ":and:f" }。
+ 查找字符出现的第一个位置：`int indexOf(String str, int fromIndex)`，查找出现的最后一个位置：`int lastIndexOf(String str, int fromIndex)`。其中fromIndex是查找开始位置。

### StringBuilder

## 高精度数字类型

## Bit位相关位

## 正则表达式

> 可以参考[正则表达式菜鸟教程](https://www.runoob.com/java/java-regular-expressions.html)

+ Java缺少类似C的scanf那样能够处理"%s->%s"的输入规格化函数。但是Java的正则表达式更加强大，事实上也不是非常难用。
+ 正则表达式相关类主要位于*Java.util.regex*包下，最常用的`Pattern`和`Matcher`类。
+ 其中`Pattern`类主要用于正则式的预处理，`Matcher`类则用模式匹配输入的字符串。

### 简单使用

+ Pattern一般使用静态方法`Pattern.complie(String regex)`获得，regex就是正则表达式
+ Matcher则使用`Pattern.matcher(String input)`获得，也可以用`Matcher.reset(CharSequence input)`重置。
+ Matcher的几个查找方法：

|      方法       | 作用                                                         |
| :-------------: | :----------------------------------------------------------- |
|    lookingAt    | 只从输入字符串头部开始查找，但不是要求匹配整个串             |
|     matches     | 匹配整个串                                                   |
|      find       | 从头部开始匹配，**找到一个匹配子串就暂停**，可以提取匹配的子串。**暂停后可以继续查找下一部分输入串**，直到查到输入串的末尾。 |
| find(int start) | 从指定位置开始find查找。                                     |

以上方法均返回boolean表示成不成功。

+ 可见最复杂也最好用的是`find`方法。

举个例子，我们输入"AB->CD;EF->GH;IJ->KL;"，我们想提取"AB CD EF GH IJ KL"则有

```java
Pattern pattern=Pattern.compile("([A-Z]+)->([A-Z]+);");
Matcher matcher=pattern.matcher("AB->CD;EF->GH;IJ->KL;");
while(matcher.find()) {
    for(int i=1;i<=matcher.groupCount();i++) {
        System.out.println(matcher.group(i));
    }
}
/*
AB
CD
EF
...
*/
```

我们想提取"AB->CD; EF->GH; IJ->KL;"则有：

```java
Pattern pattern=Pattern.compile("([A-Z]+)->([A-Z]+);");
Matcher matcher=pattern.matcher("AB->CD;EF->GH;IJ->KL;");
while(matcher.find()) {
    System.out.println(matcher.group());
}
/*
AB->CD;
EF->GH;
...
*/
```

+ 注意正则式中"(XX)"的使用，用于表示子匹配组，`group()`输出整个正则式的匹配串，`group(int i)，i>=1`输出该次匹配的第i个子匹配组。