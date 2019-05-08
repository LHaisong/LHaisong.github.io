---
layout:     post
title:      ThreadLocal
subtitle:   介绍、使用
date:       2019-05-08
author:     LHaisong
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - java并发
---

## 线程封闭
- 当访问共享变量时，往往需要加锁来保证数据同步。一种避免使用同步的方式就是不共享数据。如果仅在单线程中访问数据，就不需要同步了。这种技术称为线程封闭。在Java语言中，提供了一些类库和机制来维护线程的封闭性，例如局部变量和ThreadLocal类，本文主要深入讲解如何使用ThreadLocal类来保证线程封闭。
## ThreadLocal介绍
- ThreadLocal类能使线程中的某个值与保存值的对象关联起来，它提供了set()、get()方法，这两个方法为每个使用该变量的线程保存一个独立的副本，而且副本之间互不干扰;
- 使用场景：JDBC数据库连接池
## threadlocal存在的内存泄漏问题
- threadlocal的实现：每个thread维护一个ThreadLocalMap映射表(类似HashMap),这个映射表以ThreadLocal实例为key，value是真正需要存储的Object;也就是说ThreadLocal本身并不存放值，而是作为key让线程从ThreadLocalMap中获取value，而ThreadLocalMap与key之间的引用是弱引用，弱引用在GC时会被回收掉。
- 造成内存泄漏的原因：由于threadlocal没有外部强引用，所以在GC的时候一定会被回收，这样一来ThreadLocalMap中就存在了null key的映射;如果当前线程再迟迟不结束的话，这些key为null的Entry的value就会一直存在一条强引用链：Thread Ref -> Thread -> ThreaLocalMap -> Entry -> value永远无法回收，造成内存泄漏;起始设计者也考虑到了这种情况，所以提供了一个remove()方法来清除null key的键值对
- 但是也存在一些情况预防措施并不能保证不会造成内存泄漏：
1. 使用了static修饰的threadlocal，延长了生命周期，可能造成内存泄漏
2. 声明了threadlocal，但是并未使用过

详见：[https://blog.xiaohansong.com/ThreadLocal-memory-leak.html](https://blog.xiaohansong.com/ThreadLocal-memory-leak.html "threadlocal安全危机")

## ThreadLocal源码
[https://blog.csdn.net/forezp/article/details/77620769](https://blog.csdn.net/forezp/article/details/77620769 "ThreadLocal")
