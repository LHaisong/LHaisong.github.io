---
layout:     post
title:      HashMap、HashTable、ConcurrentHashMap
subtitle:   Map子类的宏观比较
date:       2019-05-05
author:     LHaisong
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - Collections
---
## 4 HashMap、HashTable、ConcurrentHashMap
 - 线程安全角度：HashTable是线程安全的;它内部的方法都是由synchronized修饰的;而HashMap不是，但是如果需要线程安全，则使用ConcurrentHashMap,它的效率比hashtable高很多    
 - hashmap允许putnull值，而hashtable和concurrenthashmap不允许  
 - 初始容量和扩容的区别：hashtable的初始容量是11，每次扩容容量newCap=2n+1;而hashmap和concurrenthashmap的的初始容量都是16，每次扩容容量变为原来的2倍;如果给的初始容量，hashtable直接将给定值作为容量，而   hashmap和concurrenthashmap是选择大于该值的第一个2的幂次方;  
 - 底层数据结构：hashtable不存在像链表大于阈值就转换为红黑树的操作;  
