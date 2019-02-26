---
title: Thread与Runnable的区别
date: 2019-02-26 14:44:43
categories:
 - Java
tags:
 - Java
---
Java创建线程有两种方式：
1. 继承Thread类
2. 实现Runnable接口，实际Thread类也实现了Runnable接口。

看下Thread的构造方法列表：
* public Thread()
* public Thread(Runnable target)
* public Thread(Runnable target, String name)
* public Thread(ThreadGroup group, Runnable target, String name)
* public Thread(ThreadGroup group, Runnable target, String name, long stackSize) 
* public Thread(ThreadGroup group, String name)

可以看到Runnable接口的实例是Thread的target。run犯法的实现方式是怎么样子的呢？
``` Java
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
```

这个调用的形式就是通过一个线程实例（创建了一个Thread），调用了另一个线程的run方法。

所以new Thread()即创建了一个线程。new Thread(Runnable)用一个线程作为容器，共享target的线程变量。为什么呢？

线程私有变量和线程间共有变量。通过new Thread()的方式创建线程即为线程修改私有变量。new Thread(Runnable)的线程私有变量是target，所以需要共享target的变量。

当然还有Java的单继承的原因，实际使用中使用Runnable的时候更多。

还有一点，如果手动调用Runnable.run方法是什么效果呢？不会创建一个新线程，跟普通的同步方法执行一样。
