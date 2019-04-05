---
title: HashMap之put的过程
categories:
  - Java
  - Map
tags:
  - Java
  - Map
  - HashMap
date: 2019-03-07 14:27:00
---
HashMap是Java集合类中，使用比较多的一个集合类。类似request的参数集合、jdbcTemplate的数据库返回啊，这些不定长度的key-value型数据集合，大量的使用到了HashMap。那到底HashMap的api有啥神秘呢？put的过程到底有什么问题呢？

# PUT的实现
HashMap的Put过程基本可以分为两部分，确认插入位置和更新数据。

两个过程其实奥秘就是两个基础API：hashCode和equals。

``` java
	//先计算key的hash值，调用hashCode方法，再进行位移计算。
 	public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
    //实现放入值的核心过程。
	final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //读取当前数组的长度，如果为空，初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 按照hash读取数组指定位置的元素，为空直接插入
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
