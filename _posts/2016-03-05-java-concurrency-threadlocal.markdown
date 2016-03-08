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

ThreadLocal其实是线程封闭的一种规范化的实现，它通过提供一组get和set的接口为每个使用该变量的线程保存一份独立的副本。对于那种按线程多实例（每个线程对应一个实例）的对象的访问，并且这个对象很多地方都要用到的情况（例如数据库连接管理、会话session管理以及线程私有的消息队列等），ThreadLocal就会展现出它的魅力。

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
