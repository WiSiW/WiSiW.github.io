---
title: 【读书笔记】数据结构与抽象 java语言描述
date: 2021-07-05 14:00:12
tags:
    - 读书笔记
    - 数据结构
    - Java

    
---

封装

封装：常常被称为信息隐藏。将数据和方法放在一个类中，隐藏了使用类时不需要的实现细节

抽象：是一个要求你关注什么而不是如何的过程。当设计类的时候，执行数据抽象

正确的封装将类定义为两部分：客户接口和实现

​	客户接口描述程序员使用这个类时必须了解的一切事情。包括类的公有方法的方法头、注释（告诉程序员如何使用这些公有方法）、以及类中公有定义的任何常量

​	实现部分有所有的数据域及所有方法的定义组成，包括公有、私有及保护的方法

说明方法

注释：

前置条件和后置条件

​	前置条件：是一条条件语句，在方法执行前必须为真。除非前置条件满足，否则不应该使用方法，也不能期待方法能正确执行

​	后置条件：是一条语句，当前置条件满足且完全执行方法后，它为真。一般来说，后置条件描述方法调用产生的所有影响

​		对于一个值方法，后置条件将描述方法返回的值

​		对于一个void方法，后置条件描述所做的动作及调用对象的任何修改

​	职责



说明一个包，使用UML进行表示

```
+getCurrentSize(): integer
+isEmpty(): boolean
+add(newEntry: T): boolean
+clear(): void
+remove(): T
+remove(anEntry T): T
+getFrequencyOf(anEntry: T): integer
+contains(anEntry: T): boolean
+toArray(): T[]
```



包类的java接口

```java
/**
 * An interface that describes the operations of a bag of objects
 */
public interface BagInterface<T> {
    /**
     * Gets the current number of entries in this bag.
     * @return The integer number of entries currently in the bag.
     */
    public int getCurrentSize();

    /**
     * Sees whether this bag is empty.
     * @return true if the bag is empty, or false if not.
     */
    public boolean isEmpty();

    /**
     * Adds a new entry to this bag.
     * @param newEntry The objects to be added as a new entry
     * @return True if the addition is successful, or false if not.
     */
    public boolean add(T newEntry);

    /**
     * Removes all entries from this bag.
     */
    public void clear();

    /**
     * Removes one unspecified entry from this bag, if possible.
     * @return Either the removed entry, if the removal was successful, or null.
     */
    public T remove();

    /**
     * Remove one occurrence of a given entry from this bag, if possible.
     * @param anEntry The entry to be removed.
     * @return True if the removal was successful, or false if not.
     */
    public T remove(T anEntry);

    /**
     * Counts the number of times a given entry appears in this bag.
     * @param anEntry The entry to be counted.
     * @return The number of times anEntry appears in the bag.
     */
    public int getFrequencyOf(T anEntry);

    /**
     * Tests whether this bag contains a given entry
     * @param anEntry The entry to locate.
     * @return True if the bag contains an Entry, or false if not.
     */
    public boolean contains(T anEntry);

    /**
     * Retrieves all entries that are in this bag
     * @return A newly allocated array of all the entries in the bag. Note: If the bag is empty, the returned array is empty.
     */
    public T[] toArray();
}
```

