---
layout:     post
title:      ReentrantLock和Synchronized
subtitle:   原理、区别、使用
date:       2019-05-06
author:     LHaisong
header-img: img/post-bg-re-vs-ng2.jpg
catalog: true
tags:
    - java并发
---
## 线程安全？  
> 在使用中造成线程安全的原理主要有两点：
> 1.存在共享数据  
> 2.存在多个线程对共享数据的操作  
> 自然而然我们可能需要这样一个方案，当存在多个线程对共享数据进行操作的时候使用某种手段使得在某一个时间段内只能有一个线程操作数据，当这个线程操作完成之后，其它线程才可以对数据进行操作;可以看出这是一种互斥的访问方式，这叫互斥锁。在java中互斥锁的实现可依赖于关键字Synchronized和JUC包下的并发工具。  
## 公平锁与非公平锁
- 公平锁：在获取不到锁时会进入阻塞队列，当拥有锁线程执行完毕后等待队列中的第一个线程获得锁;
- 非公平锁：在获取不到锁时会进入阻塞队列，当拥有锁线程执行完毕后等待队列中的线程都会试图去抢占锁;

## Synchronized介绍    
1.synchronized是java的一个原生关键字，用于实现互斥锁；  
2.synchronized关键字修饰的对象实现了操作的可见性、有序性；

## synchronized关键字的用法  
- synchronized关键字有3种用法：
> 修饰实例方法：作用于当前实例，在进入方法前要先获得该实例对象的锁
> 修饰静态方法：作用于当前类对象，在进入方法前要先获得该类对象的锁
> 修饰代码块：指定加锁对象，对给定对象加锁，访问时需要先获得对象的锁

- 修饰实例方法:实例方法不包含静态方法
> /**
>  * synchronized 修饰实例方法
>  */
>    public synchronized void increase(){  
>        i++;  
>    }  

- 修饰静态方法：static关键字
> /**
> * 作用于静态方法,锁是当前class对象,也就是  
> * AccountingSyncClass类对应的class对象  
> */  
> public static synchronized void increase(){  
>    i++;  
> }  
> 
> /**  
> * 非静态,访问时实例对象锁不一样不会发生互斥  
> */  
> public synchronized void increase4Obj(){  
>    i++;  
> }  

- 修饰代码块：为指定的对象加锁，在方法体比较大的时候，如果需要加锁的代码块只是很小的一部分，则应该采用这种方式
>单例模式的实现
>class Singleton{  
>  private volatile Singleton instance;  
>  private static Singleton getInstance(){  
>    if(instance==null){  
>       synchronized(Singleton.class){  
>          if(instance==null){  
>             instance=new Singleton();  
>          }  
>       }  
>     }  
>     return instance;  
>   }  
>}  

## Synchronized底层实现原理
> 在Java虚拟机中Synchronized关键字的实现是通过进入和退出Monitor对象实现的，无论是显示(有明确的monitorenter和monitorexit指令)还是隐式同步都是如此;需要注意的是，当synchronized修饰方法时，是由方法调用指令读取运行常量池中的ACC_SYNCHRONIZED标志隐式实现的;
> 
### 要理解synchronized的原理必须要了解java的对象头结构
- 在java虚拟机中java对象在内存中的存储可分为三块区域：对象头、实例数据、对齐填充
- 对象头包含两个部分：
> 第一部分是用于存储对象自身运行时数据，如hash码、GC分代年龄、锁状态标志、线程持有锁、偏向线程ID，这部分数据也被称为Mark Word
![](https://i.imgur.com/htraNFz.png)
> 另外一部分是类型指针，即是对象指向它的类的元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。并不是所有的虚拟机实现都必须在对象数据上保留类型指针，换句话说查找对象的元数据信息并不一定要经过对象本身。另外，如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据，因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是从数组的元数据中无法确定数组的大小。  

-对于Synchronized来说，它是一个重量级锁，它的指针10指针指向一个monitor对象的起始地址，每个对象都有一个Monitor对象与它对应;
-在虚拟机中对象与monitor之间的实现由多种方式；monitor可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，但当一个 monitor 被某个线程持有后，它便处于锁定状态。在Java虚拟机(HotSpot)中，monitor是由ObjectMonitor实现的，其主要数据结构如下（位于HotSpot虚拟机源码ObjectMonitor.hpp文件，C++实现的）
    ObjectMonitor() {  
    _header       = NULL;  
    _count        = 0; //记录个数  
    _waiters      = 0,  
    _recursions   = 0;  
    _object       = NULL;  
    _owner        = NULL;  
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet  
    _WaitSetLock  = 0 ;  
    _Responsible  = NULL ;  
    _succ         = NULL ;  
    _cxq          = NULL ;  
    FreeNext      = NULL ;  
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表  
    _SpinFreq     = 0 ;  
    _SpinClock    = 0 ;  
    OwnerIsThread = 0 ;  
  }  
- ObjectMonitor中有两个队列，_WaitSet 和 _EntryList，用来保存ObjectWaiter对象列表( 每个等待锁的线程都会被封装成ObjectWaiter对象)，_owner指向持有ObjectMonitor对象的线程，当多个线程同时访问一段同步代码时，首先会进入 _EntryList 集合，当线程获取到对象的monitor 后进入 _Owner 区域并把monitor中的owner变量设置为当前线程同时monitor中的计数器count加1，若线程调用 wait() 方法，将释放当前持有的monitor，owner变量恢复为null，count自减1，同时该线程进入 WaitSe t集合中等待被唤醒。若当前线程执行完毕也将释放monitor(锁)并复位变量的值，以便其他线程进入获取monitor(锁)。
![](https://i.imgur.com/89E4wOW.png)

## Synchronized代码块底层原理  
- 通过一段代码的反编译来理解  
     public static void increase() {  
		synchronized (SyncTest.class) {  
			i++;  
		}  
	}  
反编译的结果如下：  
![](https://i.imgur.com/XNultME.png)  
从图中我们可以看出，在进入代码块和出代码块时分别有monitorenter和monitorexit修饰，当执行monitorenter时，当前线程会尝试获取object对应的monitor对象的持有权，如果object的monitor的进入计数器为0，则线程可以成功进入monitor，并将计数器设置为1，取锁成功。如果当前线程已经拥有 objectref 的 monitor 的持有权，那它可以重入这个 monitor (关于重入性稍后会分析)，重入时计数器的值也会加 1。倘若其他线程已经拥有 objectref 的 monitor 的所有权，那当前线程将被阻塞，直到正在执行线程执行完毕，即monitorexit指令被执行，执行线程将释放 monitor(锁)并设置计数器值为0 ，其他线程将有机会持有 monitor 。值得注意的是编译器将会确保无论方法通过何种方式完成，方法中调用过的每条 monitorenter 指令都有执行其对应 monitorexit 指令，而无论这个方法是正常结束还是异常结束。为了保证在方法异常完成时 monitorenter 和 monitorexit 指令依然可以正确配对执行，编译器会自动产生一个异常处理器，这个异常处理器声明可处理所有的异常，它的目的就是用来执行 monitorexit 指令。从字节码中也可以看出多了一个monitorexit指令，它就是异常结束时被执行的释放monitor 的指令。

## Synchronized方法实现原理  
- 通过一段代码学习  
    public static int i=0;  
	public  static synchronized void increase() {  
		i++;  
	}  
- 反编译的结果如下：  
![](https://i.imgur.com/xQpDoog.png)   
- 从图中我们可以看出synchronized修饰方法是通过方法调用指令读取常量池中的ACC_SYCHRONIZED标记实现的，该标识指明了该方法是一个同步方法，JVM通过该ACC_SYNCHRONIZED访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。这便是synchronized锁在同步代码块和同步方法上实现的基本原理。

## Synchronized和ReentrantLock的比较：
- 相同点：
> 两者都是可重入锁，即当线程A获取对象的锁后，再次请求获取对象的锁是能够取锁成功的，获取锁的同时计数器的值依然要+1，在释放的时候，同样需要将计数器的值减小为0的时候线程A才能释放锁；
> 需要注意的是两者实现可重入锁的原理是不一样的，对于synchronized来说，当计数器为0时表示没有线程获取锁，当线程获取到锁后jvm后记下这个线程，并将锁计数器+1；此时其它线程请求该锁，则必须等待；而该持有锁的线程如果再次请求这个锁，就可以再次拿到这个锁，同时计数器会递增；当线程退出同步代码块时，计数器会递减，如果计数器为 0，则释放该锁。对于ReentrantLock来说，ReentrantLock在请求获取锁时会进行一个判断，判断当前是否有线程持有锁，持有锁的线程是否是当前请求锁的线程，如果锁的Owner是当前请求的线程，则会获取到锁，并将锁计数器+1；如果锁的Owner不是当前请求的线程，当前线程任然会试图抢占锁。
- 不同点：
1. 实现上：synchronized由jvm实现，而ReentrantLock由jdk实现  
1. 性能上：在jvm未对synchronized关键字做优化之前差ReentrantLock很多，但是synchronized引入偏向锁、轻量级锁、和自旋以后两者在性能上就差别不大了   
1. 功能上：synchronized的实现更简单，加锁和解锁的工作由jvm来做，而ReentrantLock则需要用户手动的进行加锁和释放锁，为避免遗忘释放锁而造成阻塞在finally语句中完成释放锁是不错的选择  
1. ReenTrantLock独有的能力：  
   ReentrantLock可以指定是非公平锁(默认)or公平锁，而synchronized只能是非公平锁。  
   ReenTrantLock提供了一个Condition（条件）类，用来实现分组唤醒需要唤醒的线程们，而不是像synchronized要么随机唤醒一个线程要么唤醒全部线程(唤醒机制：synchronized依赖于Object类中的notify()和notifyAll方法)。  
   ReenTrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。  
