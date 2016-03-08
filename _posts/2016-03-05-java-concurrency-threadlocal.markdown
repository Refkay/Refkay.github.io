---
layout:     post
title:      "JAVA并发学习之ThreadLocal"
date:       2016-03-05 14:00:00
author:     "Lemon Yang"
tags:
    - JAVA
    - 并发编程
---

>关于java并发编程的相关文章都是阅读了《java并发编程实战》之后的读书笔记总结

### 概述

ThreadLocal其实是线程封闭的一种规范化的实现，它通过提供一组get和set的接口为每个使用该变量的线程保存一份独立的副本。__对于那种按线程多实例（每个线程对应一个实例）的对象的访问，并且这个对象很多地方都要用到的情况（例如数据库连接管理、会话session管理以及线程私有的消息队列等），ThreadLocal就会展现出它的魅力__。

下面的这个小例子展示了ThreadLocal的常规使用：

	/**
 	* 线程资源类
 	* 这里只有简单的两个标志线程属性的变量,我们还可以往里面增添其他线程的属性(一般是我们对应于那个线程都要使用并且各不相同的属性)
 	*/
	public class ThreadResource {
    
    	private String threadName;
    	private int threadId;

    	public ThreadResource(int threadId,String threadName){
        	this.threadId=threadId;
        	this.threadName=threadName;
    	}

    	public String getThreadName() {
        	return threadName;
    	}

    	public void setThreadName(String threadName) {
        	this.threadName = threadName;
    	}

    	public int getThreadId() {
        	return threadId;
    	}

    	public void setThreadId(int threadId) {
        	this.threadId = threadId;
    	}
	}
	
	public class Test {
    	//以一个静态实例的方式持有一个ThreadLocal对象,它里面以map的形式存储了线程的局部变量
    	static ThreadLocal<ThreadResource> resoursePackage=new ThreadLocal<ThreadResource>(){
        	@Override
        	protected ThreadResource initialValue() {
            	return new ThreadResource(0,"initialThread");
        	}
    	};

    	private static class TestThread extends Thread {
        	@Override
        	public void run(){
           		resoursePackage.set(new ThreadResource(1,"testThread"));
            	System.out.println(resoursePackage.get().getThreadName()+ resoursePackage.get().getThreadId());
        	}
    	}

    	public static void main	(String[] args){
        	System.out.println(resoursePackage.get().getThreadName()+ resoursePackage.get().getThreadId());
        	new TestThread().start();
    	}
	}

### 源码解析ThreadLocal的实现

* __get()方法的实现__

    	/**
     	* Returns the value in the current thread's copy of this
     	* thread-local variable.  If the variable has no value for the
     	* current thread, it is first initialized to the value returned
     	* by an invocation of the {@link #initialValue} method.
     	* 
     	* @return the current thread's value of this thread-local
     	*/
    	public T get() {    	
    		//获取当前threadlocal变量所属的线程
        	Thread t = Thread.currentThread();
        	//根据线程获取到一个ThreadLocalMap的对象
        	ThreadLocalMap map = getMap(t);
        	//如果线程已经绑定了一个ThreadLocalMap对象的话，则从中获取到里面所保存的值，否则使用初始化的值
        	if (map != null) {
            	ThreadLocalMap.Entry e = map.getEntry(this);
            	if (e != null) {
                	@SuppressWarnings("unchecked")
                	T result = (T)e.value;
                	return result;
            	}
        	}
        	return setInitialValue();
    	}
    	
    我们看一下上面_getMap()_的方法的实现
    
        /**
     	* Get the map associated with a ThreadLocal. Overridden in
     	* InheritableThreadLocal.
     	*
     	* @param  t the current thread
     	* @return the map
     	*/
    	ThreadLocalMap getMap(Thread t) {
    		//返回线程的一个ThreadLocalMap的成员变量,下面是该成员变量在threa类中的声明
    		//ThreadLocal values pertaining to this thread. This map is maintained by the ThreadLocal class.
    		//ThreadLocal.ThreadLocalMap threadLocals = ull;
        	return t.threadLocals;
    	}
    	
    再看一下线程尚未绑定ThreadLocalMap对象的时候，调用的_setInitialValue（）_的方法的实现
    
        /**
     	* Variant of set() to establish initialValue. Used instead
     	* of set() in case user has overridden the set() method.
     	*
     	* @return the initial value
     	*/
    	private T setInitialValue() {
    		//initialValue（）就是我们在新建一个ThreadLocal变量的时候，实现的protected的那个方法。
        	T value = initialValue();
        	Thread t = Thread.currentThread();
        	ThreadLocalMap map = getMap(t);
        	if (map != null)
            	map.set(this, value);
        	else
        		//由此我们可知，线程尚未绑定到ThreadLocalMap对象的时候
        		//ThreadLocal为我们使用在initialValue设置的值初始化了一个对象值，并绑定到该线程上。
            	createMap(t, value);
        	return value;
    	}
	
	在上面的代码中，我们一直提及到了一个ThreadLocalMap的类，它其实是在ThreadLocal的一个静态内部类，它是一个ThreadLocal自定义的hash map对象，用于保存线程的局部变量。在它里面，又包含了一个Entry的静态内部类，它里面就是对应的所要存储的值。我们看一下源码的实现
	
	    /**
     	* ThreadLocalMap is a customized hash map suitable only for
     	* maintaining thread local values. No operations are exported
     	* outside of the ThreadLocal class. The class is package private to
     	* allow declaration of fields in class Thread.  To help deal with
     	* very large and long-lived usages, the hash table entries use
     	* WeakReferences for keys. However, since reference queues are not
     	* used, stale entries are guaranteed to be removed only when
     	* the table starts running out of space.
     	*/
    	static class ThreadLocalMap {

        	/**
         	* The entries in this hash map extend WeakReference, using
         	* its main ref field as the key (which is always a
         	* ThreadLocal object).  Note that null keys (i.e. entry.get()
         	* == null) mean that the key is no longer referenced, so the
         	* entry can be expunged from table.  Such entries are referred to
         	* as "stale entries" in the code that follows.
         	*/
        	static class Entry extends WeakReference<ThreadLocal<?>> {
            	/** The value associated with this ThreadLocal. */
            	Object value;
				//以ThreadLocal对象作为键值，保存threadlocal变量所包含的值
				//我们在ThreadLocal的get方法当中也是根据threadlocal变量取出所存储的值
            	Entry(ThreadLocal<?> k, Object v) {
                	super(k);
                	value = v;
            	}
        	}
        ｝
     
* __set()方法的实现__

		/**
     	* Sets the current thread's copy of this thread-local variable
     	* to the specified value.  Most subclasses will have no need to
     	* override this method, relying solely on the {@link #initialValue}
     	* method to set the values of thread-locals.
     	*
     	* @param value the value to be stored in the current thread's copy of
     	*        this thread-local.
     	*/
     	
    	public void set(T value) {
    		//获取threadlocal对象所属的线程
        	Thread t = Thread.currentThread();
        	//获取线程所绑定的ThreadLocalMap对象
        	ThreadLocalMap map = getMap(t);
        	if (map != null)
            	map.set(this, value);
        	else
        		//在线程尚未初始化并绑定ThreadLocalMap对象的时候，使用给定的value值新建一个，并将线程与该对对象关联起来
            	createMap(t, value);
    	}
    	
在阅读了上面的源码之后，我们大概已经明白了ThreadLocal是怎么做到为每个线程保存一份引用对象的拷贝的值的。每个thread对象都会持有一个ThreadLocal.ThreadLocalMap的对象的引用，而我们通过ThreadLocal对象找到了线程所持有的这个ThreadLocalMap对象，并往其中添加、移除或获得我们所要保存的引用对象的值。

关于ThreadLocalMap里面实现的自定义的hash map我们可以在ThreadLocal的源码中深入了解，这里不做进一步的深入。