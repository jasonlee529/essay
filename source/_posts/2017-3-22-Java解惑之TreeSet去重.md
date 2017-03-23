---
title: Java解惑之TreeSet是如何去重的
date: 2017-03-22 13:53:36
categories:
 - Java
tags:
 - Java
 - Collection
 - Set
 - TreeSet
---
>引言： 最近在处理一个问题，大致是这个样子，从数据库里面取出一个集合，取出来的数据放到一个JavaBean里面。结果得到的集合长度为1.

TreeSetSet的一个实现，默认实现排序；故TreeSet的泛型类型必须是Comparable或者Comparator。TreeSet基于TreeMap实现。

![TreeSet类图](http://oi4qj3o0c.bkt.clouddn.com/essay/treeset/class-diagram.png)
<!-- more -->
## 实例
``` java
public class Person implements Comparable<Person>{
    private String name;
    private int order;
    public Person(String name , int order){
        this.name = name;
        this.order = order;
    }
    public static void main(String [] args){
        TreeSet<Person> users = new TreeSet<Person>();
        users.add(new Person("Jason Kidd",1));
        users.add(new Person("Kobe Bryant",1));
        users.add(new Person("Jasme Harden",1));
        users.add(new Person("Leborn James",1));
        System.out.println(" size= "users.size());
    }
    // getter and setter
}
size= 1
```
### 认知被改写
上面代码的结果应该是4,因为我们没有重写hashCode和equals方法啊，Set的去重逻辑变了吗？

重写上面的例子：
``` java
public class Person implements Comparable<Person>{
    public static void main(String [] args){
        TreeSet<Person> users = new TreeSet<Person>();
        users.add(new Person("Jason Kidd",1));
        users.add(new Person("Kobe Bryant",2));
        users.add(new Person("Jasme Harden",1));
        users.add(new Person("Leborn James",1));
        System.out.println(" size= "users.size());
    }
    // getter and setter
}
size= 2
```
表面看好像是compareTo方法觉得了去重；为了证实这个猜测，进入下面的代码里面去证实下吧。
## 源码分析
### 理论依据
``` java
 * <p>Note that the ordering maintained by a set (whether or not an explicit
 * comparator is provided) must be <i>consistent with equals</i> if it is to
 * correctly implement the {@code Set} interface.  (See {@code Comparable}
 * or {@code Comparator} for a precise definition of <i>consistent with
 * equals</i>.)  This is so because the {@code Set} interface is defined in
 * terms of the {@code equals} operation, but a {@code TreeSet} instance
 * performs all element comparisons using its {@code compareTo} (or
 * {@code compare}) method, so two elements that are deemed equal by this method
 * are, from the standpoint of the set, equal.  The behavior of a set
 * <i>is</i> well-defined even if its ordering is inconsistent with equals; it
 * just fails to obey the general contract of the {@code Set} interface.
 //翻译
注意，如果要正确实现 Set 接口，则 set 维护的顺序（无论是否提供了显式比较器）必须与 equals 一致。
（关于与 equals 一致 的精确定义，请参阅 Comparable 或 Comparator。）这是因为 Set 接口是按照
 equals 操作定义的，但 TreeSet 实例使用它的 compareTo（或 compare）方法对所有元素进行比较
 ，因此从 set 的观点来看，此方法认为相等的两个元素就是相等的。即使 set 的顺序与 equals 不一致，
  其行为也是 定义良好的；它只是违背了 Set 接口的常规协定。
```
上面提到了TreeSet的equal方法是依据Comparable或者Comparator接口判断的，和前面测试的结果一致。

### 源码解读：
代码流转：

![TreeSet add](http://oi4qj3o0c.bkt.clouddn.com/essay/java/collection/treesetTreeSet-add-sequence.png
)

前面提过了，TreeSet是底层是基于TreeMap存储数据的：
``` java
public class TreeSet{
  public TreeSet() {
      this(new TreeMap<E,Object>());
  }
}
```
构造TreeSet的时候，并没有检查参数类型必须是Comparable或者Comparator的实现类。

``` java
public class TreeSet{
  private static final Object PRESENT = new Object();
  public boolean add(E e) {
     return m.put(e, PRESENT)==null;
  }
}
```
添加元素的操作，实际就是如何给TreeMap增加元素的操作。
``` java
// java.util.TreeMap.java
public V put(K key, V value) {
  Entry<K,V> t = root;
  if (t == null) {
      compare(key, key); // type (and possibly null) check
      root = new Entry<>(key, value, null);
      size = 1;
      modCount++;
      return null;
  }
  int cmp;
  Entry<K,V> parent;
  // split comparator and comparable paths
  Comparator<? super K> cpr = comparator;
  if (cpr != null) {
      do {
          parent = t;
          cmp = cpr.compare(key, t.key);
          if (cmp < 0)
              t = t.left;
          else if (cmp > 0)
              t = t.right;
          else
              return t.setValue(value);
      } while (t != null);
  }
  else {
      if (key == null)
          throw new NullPointerException();
      @SuppressWarnings("unchecked")
          Comparable<? super K> k = (Comparable<? super K>) key;
      do {
          parent = t;
          cmp = k.compareTo(t.key);
          if (cmp < 0)
              t = t.left;
          else if (cmp > 0)
              t = t.right;
          else
              return t.setValue(value);
      } while (t != null);
  }
  Entry<K,V> e = new Entry<>(key, value, parent);
  if (cmp < 0)
      parent.left = e;
  else
      parent.right = e;
  fixAfterInsertion(e);
  size++;
  modCount++;
  return null;
}
final int compare(Object k1, Object k2) {
   return comparator==null ? ((Comparable<? super K>)k1).compareTo((K)k2)
       : comparator.compare((K)k1, (K)k2);
}
```
上面一段看起来很长，我们重看其中几句就好了
* TreeMap内置默认的Comparator比较器，一旦设定，后续所有的比较必须依据此比较器进行;发现不符合比较器方式的，抛出异常。
* 核心的两个do-while循环，依据比较器结果变量__cmp__，判断是将值插入的觉提位置。一旦__cmp=0__将value放到相同位置。

相同的key会覆盖，所有前面测试的时候，集合长度不增加，主要因为compareTo方法返回了0.
