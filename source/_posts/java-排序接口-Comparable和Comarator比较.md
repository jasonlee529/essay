---
title: Java 排序接口 Comparable和Comarator比较
date: 2017-02-23 11:01:49
categories:
  - Java
tags:
  - Java
  - 算法
---
## java 排序Comparable和Comparator使用
java提供了两个排序用的接口Comparable和Comparator,一般情况下使用区别如下：
> 1. Comparable 接口用于类的固定排序方式上面，比如类实现Comparable接口，实现compareTo方法，
做为类默认排序实现。
> 2. Comprator接口通常用于特殊场景下面的排序方式，比如学生成绩在计算过程中需要按照不同科目排序一样。

无论实现哪个接口，都可以使用Collections.sort方法对集合或者数组进行排序。
```java
public class Collections {
  public static <T extends Comparable<? super T>> void sort(List<T> list) {
        list.sort(null);
    }
   public static <T> void sort(List<T> list, Comparator<? super T> c) {
        list.sort(c);
    }
  }
```
<!-- more -->

可以看到，具体实现由List的sort方法解决。
```java
public class List{
 default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, (Comparator) c);
        ListIterator<E> i = this.listIterator();
        for (Object e : a) {
            i.next();
            i.set((E) e);
        }
    }
}
```
没有搞明白为什么方法是default的包权限？
list直接把数据转换为数组，转换为数据排序实现。
```java
public class Arrays{
    public static <T> void sort(T[] a, Comparator<? super T> c) {
        if (c == null) {
            sort(a);
        } else {
            if (LegacyMergeSort.userRequested)
                legacyMergeSort(a, c);
            else
                TimSort.sort(a, 0, a.length, c, null, 0, 0);
        }
    }
    public static void sort(Object[] a) {
            if (LegacyMergeSort.userRequested)
                legacyMergeSort(a);
            else
                ComparableTimSort.sort(a, 0, a.length, null, 0, 0);
        }
}
```
处理过程中都出现legencyMergeSort，jdk8中已经标记为将要移除的算法，mergeSort就是归并排序。
严格来讲，TimSort也是归并排序的一种优化版本，具体的算法实现方式最早来源于python的实现。有兴趣可以搜索下
资料。
> 归并排序:将2个或者多个有序集合合并为一个更大的有序集合。
具体做法；将长度为n的数组看做是n个数据，两两合并为一个长度为2的有序集合；重复前面的步骤，直到得到一个长度为n的数组为止。

简单来讲MergeSort需要把数组分治，利用第三个数据来承载合并后的数据，这样可能产生大量的数组对象。
TimSort是将归并的过程在本地实现。这部分还没有搞懂，搞定后再继续。
```java
public  class TimSort{
    static <T> void sort(T[] a, int lo, int hi, Comparator<? super T> c,
                         T[] work, int workBase, int workLen) {
        assert c != null && a != null && lo >= 0 && lo <= hi && hi <= a.length;
        int nRemaining  = hi - lo;
        if (nRemaining < 2)
            return;
        if (nRemaining < MIN_MERGE) {
            int initRunLen = countRunAndMakeAscending(a, lo, hi, c);
            binarySort(a, lo, hi, lo + initRunLen, c);
            return;
        }
        TimSort<T> ts = new TimSort<>(a, c, work, workBase, workLen);
        int minRun = minRunLength(nRemaining);
        do {
            int runLen = countRunAndMakeAscending(a, lo, hi, c);
            if (runLen < minRun) {
                int force = nRemaining <= minRun ? nRemaining : minRun;
                binarySort(a, lo, lo + force, lo + runLen, c);
                runLen = force;
            }
            ts.pushRun(lo, runLen);
            ts.mergeCollapse();
            lo += runLen;
            nRemaining -= runLen;
        } while (nRemaining != 0);
        assert lo == hi;
        ts.mergeForceCollapse();
        assert ts.stackSize == 1;
    }
}
```
