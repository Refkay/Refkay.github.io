---
layout:post
date:2015-12-21 01:00:00
author:"Lemon Yang"
title:"安卓设计模式学习笔记－观察者模式"
tags:
	-design pattern
	-学习笔记
	
---

>你不会找到路，除非你敢于迷路

今天，我们谈一下23种设计模式之中的**观察者模式**

### 定义

观察者模式，顾名思义，其实就是一个对象时刻监听另一个对象的状态。这种监听是通过对象与对象之间的依赖关系实现的。

### UML类图结构


### JAVA简单实现
下面这个例子是通过java代码简单的模拟了我们平时使用“应用商店”下载应用的时候的场景。众所周知，当我们在应用商店中开始下载一个应用的时候，所有包含有这个应用信息的页面都会更新ui界面显示，下面这个只是一个粗陋的场景模拟

	/**
 	* 这个是被观察的对象,继承了java自身的Observable类
 	* Observable类当中管理了一个observer的列表以及自身的状态,通过notifyObservers这个方法将改变暴露给注册在其中的observer
 	* Created by lemon on 15/12/21.
 	*/
	public class Poster extends Observable{
    	public void doPost(){
        	//标示被观察者的被监听状态发生了改变
        	setChanged();
        	//遍历成员变量中的观察者的列表,通知自身的改变
        	notifyObservers(new String("应用开始下载"));
    	}
	}

	/**
 	* 这个是观察者类,实现了java自身的observer接口(update()方法)
 	* 当被观察的对象状态发生改变的时候,会调用其列表中的observer对象的update方法
 	* Created by lemon on 15/12/21.
 	*/
	public class Subsciber implements Observer{

    	private String mName;

    	public Subsciber(String name){
        	this.mName=name;
    	}

    	@Override
    	public void update(Observable o, Object arg) {
        	System.out.println(mName+"收到了"+arg+"的通知,开始进行界面处理");
    	}

	}
	
	/**
 	* Created by lemon on 15/12/21.
 	*/
	public class TestObserverPattern {

    	public static void main(String[] args){
        	//创建一个被观察的对象
        	Poster poster=new Poster();
        	//创建观察者
        	Subsciber downloadPager=new Subsciber("下载页");
        	Subsciber detailPager=new Subsciber("详情页");
        	Subsciber searchPager=new Subsciber("搜索页");
        	//将观察者绑定到被观察的对象
        	poster.addObserver(downloadPager);
        	poster.addObserver(detailPager);
        	poster.addObserver(searchPager);
        	//被观察的对象更新状态
        	poster.doPost();
    	}
	}
	
上面代码的运行结果如下：

![](/img/in-post/2015-12-21-observerpattern-java-sample-result.png)

在上面的代码中，我直接使用了java自身的Observer接口和Observable对象，这两个源文件都比较简单，大家可以自行查看。这里附上Observable的代码结构：

![](/img/in-post/desing_pattern_observer_observable.png)

我在一开始就提及到，观察者的模式的实现，其实就是利用对象的依赖关系。我们在Observable的源文件中可以看到，Observable对象维持来一个Observer集合的成员变量，通过这个成员变量来向每个监听者发送状态更新消息

### 观察者模式在android中的运用