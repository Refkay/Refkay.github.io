---
layout:     post
title:      "JAVA并发学习之volatile"
date:       2016-03-05 10:00:00
author:     "Lemon Yang"
tags:
    - JAVA
    - 并发编程
---

>关于java并发编程的相关文章都是阅读了《java并发编程实战》之后的读书笔记总结,另外本文还参考和引用了[Java 理论与实践: 正确使用 Volatile 变量](http://www.ibm.com/developerworks/cn/java/j-jtp06197.html)

在java的锁机制中（例如synchronized），主要包含了两种特性，即原子性（互斥）和可见性。原子性即一次只允许一个线程能够持有某个特定的锁，并访问其代码块。因此原子性可以用于实现对共享数据对协调访问，一次只有一个线程可以访问其共享对象。

### volatile保证可见性

java中的volatile关键字可以被认为是一种轻量级的synchronized机制，访问volatile变量的时候并不会执行加锁操作，不会导致线程阻塞，同时它又和锁机制一样，都具备可见性的特性（但是并不具备原子特性）。当把变量生命为volatile类型后，编译器与运行时都会注意到这个变量是共享的，因此不会把这个变量上的操作与其他内存操作一起重排序。volatile变量不会被缓存在寄存器或者其他处理器不可见的地方。因此在读取volatile变量的时候总是会返回最新写入的值。

### volatile提供线程安全

volatile变量可以用于提供线程安全，但是__必须同时满足__以下两个条件:

* __对变量的写操作不依赖于当前的值__

* __该变量没有包含在具有其他变量的不变式之中__


### 正确使用volatile关键字

在使用volatile关键字的时候，要始终牢记一个原则－－__只有在状态真正独立于程序内其他内容时才能使用 volatile__。

* __状态标识__

	volatile变量可以作为一个布尔状态标识，用于指示发生一次重要性时间，例如停止线程等
	
		class Test {
			static volatile boolean isRunning;
			
			private static class TestThread extends Thread {
				public void run(){
					while(!isRunning){
						System.out.println("the thread is still running");
					}
				}
			}
			
			public static void main	(String[] args){
				new TestThread().start();
				try {
            		Thread.sleep(2000);
        		} catch (InterruptedException e) {
            		e.printStackTrace();
        		}
				isRunning=true;
			}		
		}
		
	在上面的例子中，我们在isRunning这个布尔变量前面加上了volatile进行修饰，以确保在main线程当中对它的修改能够被我们开启的子线程所看到。（如果我们没有添加volatile关键字，有可能子线程会持续运行而看不到isRunning的值的改变）
	
* __一次性安全发布__

	在缺乏同步的情况下，可能会遇到某个对象引用的更新值（由另一个线程写入）和该对象状态的旧值同时存在。通过使用volatile关键字，我们可以实现安全发布对象。
	
		public class BackgroundFloobleLoader {
    		public volatile Flooble theFlooble;
    		
    		public void initInBackground() {
        		// do lots of stuff
        		theFlooble = new Flooble();  // this is the only write to theFlooble
    		}
		}

		public class SomeOtherClass {
    		public void doWork() {
        		while (true) { 
            		// do some stuff...
            		// use the Flooble, but only if it is ready
            		if (floobleLoader.theFlooble != null) 
                		doSomething(floobleLoader.theFlooble);
        		}
    		}
		}
		
	__⚠️上面的安全发布的必要条件是：被发布的对象必须是线程安全的，或者是有效的不可变对象（有效不可变意味着对象的状态在发布之后永远不会被修改）。volatile 类型的引用可以确保对象的发布形式的可见性，但是如果对象的状态在发布后将发生更改，那么就需要额外的同步。__

* __独立观察__

	定期发布观察结果供程序内部使用。如下面的例子所示：
	
		public class UserManager {
    		public volatile String lastUser;
    		
    		public boolean authenticate(String user, String password) {
        		boolean valid = passwordIsValid(user, password);
        		if (valid) {
            		User u = new User();
            		activeUsers.add(u);
            		lastUser = user;
        		}
        		return valid;
    		}
		}
		
	__⚠️上面这种用例要求被发布的值是有效不可变的 —— 即值的状态在发布后不会更改__
	
* __开销较低的读－写锁策略__

	如果对于某个变量的值的读操作远远超过写操作，我们可以通过将内置锁（帮助我们实现操作的原子性）和volatile关键字相结合，实现开销较低的读－写锁。
	
		@ThreadSafe
		public class CheesyCounter {
    		// Employs the cheap read-write lock trick
    		// All mutative operations MUST be done with the 'this' lock held
    		@GuardedBy("this") private volatile int value;
    		public int getValue() { return value; }
    		
    		public synchronized int increment() {
        		return value++;
    		}
		}	


		
		
		

