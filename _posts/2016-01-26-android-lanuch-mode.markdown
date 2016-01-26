---
layout:	post
title:		"安卓activity的启动模式"
date:		2016-01-26 10:00:00
author:	"Lemon Yang"
tags:
	-android
---

## 概述

一个task（任务）是当完成一个明确的工作的时候与用户交互的一系列的activity的集合。这个集合里面所有的activity都按照被打开的顺序规划在一个栈里面（这就是后退栈，是一个后进先出栈）。

设备的主页是大多数任务被创建的地方。当用户点击桌面上的一个应用图标的时候，这个应用的任务就会被调到前台。如果这个应用的任务不存在（该应用最近并没有被启动过），那么一个新的任务就会被创建，并且这个应用带有mainaction的activity就会被打开并且放到这个任务栈作为根元素。

当用户启动另一个activity的时候，新启动的这个activity就会被放置在这个栈的顶部并取得焦点。之前启动的那个activity还依然存在，但是它已经不可见并且回调了自己的onstop方法。当用户点击后退键的时候，顶部的这个activity就会被弹出（并且这个activity会被销毁），而先前的activity就会重新可见。具体可以参考下面这张官方文档里面的图片

![](/img/in-post/diagram_backstack.png)

当用户开启了一个新的任务，或者回到了主页的时候，先前的这个任务就会被作为一个整体调到后台，而这个任务栈里的所有activity都会回调它们的onstop方法，但是这个任务栈依然保持完好，它只是失去了变成了不可见的状态 。

安卓提供了activity的启动模式来管理这些任务和后退栈。

## 词语解释

1. **任务(task)**.就如上面所说，一个任务就是一系列与用户交互的activity的集合。
2. **taskAffinity**.一个任务中的每个acitity具备的类属关系（相当于这个task的名称）。可以通过在mainfest的activity的标签里面指定这一个属性值。凡是这个标签的值相同的activity在启动的时候都会被放置到相同的一个任务中。一个任务的task的类属（task的名字）是由这个任务栈的根元素决定的。默认的情况下，同一个应用下的所有activity都会具备相同的taskAffinity属性。task的名称会决定两件事情。第一个是这个activity被重新调到前台（re-parented）的时候所属的任务栈,另一个是当activit以***FLAG_ACTIVITY_NEW_TASK***这个标签被启动的时候，存放这个acitity的任务栈。(*因此这个属性通常和singleTask或者allowTaskReparenting结合使用，具体效果可以参考下面的解释*)
3. **allowTaskReparenting**.这个属性值指定了当一个activity下次被调到前台的时候是否能够从启动它的那个任务栈中移到它的taskAffinity标签所指定的任务栈中。因为具备***singleTask***和***singleInstance***启动模式的activity都只能被放置在任务栈的根部，因此re－parenting动作仅局限于***standard***和***singleTop***模式启动的activity。

## activity的启动模式

设置activity的启动模式可以通过两种方式实现：

1. 通过在mainfest菜单中的activity标签中指定lanuchMode属性，属性值可以取***standard***、***singleTop***、***singleTask***、***singleInstance***四种；
2. 通过在java代码中以intent的模式启动activity的时候设定tag标签，例如：***FLAG_ACTIVITY_NEW_TASK***、***FLAG_ACTIVITY_SINGLE_TOP***、***FLAG_ACTIVITY_CLEAR_TOP***.

当同时通过两种模式启动activity的时候，第二种方法的设定会覆盖第一种方法的设定，也就是说，第二种方法的优先级更加高。

#### standard(标准模式)

当用户对启动模式不做任何设定的时候，系统默认就会采取折中模式启动activity。在这种模式下，每次启动一个activity的时候都会创建它的一个实例，不管这个实例是否已经存在。下面这张图将会帮助你理解这种模式：

![](/img/in-post/standard.png)

### singleTop(栈顶复用模式)

当用户使用这种模式启动activity的时候，如果要启动的activity已经存在栈顶的时候，系统将不会创建这个activity的实例，只会回调该activity的onNewIntent的方法。但是如果activity不在栈顶（哪怕该activity已经存在任务栈的其它位置），系统都会再创建一个这个activity的实例。下面这张图可以帮助你理解这种启动模式。（通过intent的flag标签***FLAG_ACTIVITY_SINGLE_TOP***可以达到相同效果）


![](/img/in-post/singletop.png)


### singleTask(栈内复用模式)

当用户以这种模式启动activity的时候，系统首先会寻找该activity所需要的任务栈是否存在，如果不存在则新建一个任务栈，并把该activity放到栈中。如果任务栈已经存在的话，则看任务栈中是否已经存在这个activity的实例，如果不存在相应的实例的话，那么新建这个activity的实例，并将其压进任务栈中；如果任务栈中已经存在这个activity的实例，那么将该实例调到栈顶（这时候，在这个activity前面的activity实例都会被弹出栈）。下面这张图可以帮助你理解这种模式。（通过intent的flag标签***FLAG_ACTIVITY_NEW_TASK***可以达到相同效果，一般和标签***FLAG_ACTIVITY_CLEAR_TOP***同时使用）

![](/img/in-post/singletask.png)


### singleInstance（单实例模式）

这是singleTask模式的加强版本。它除了具备singleTask所有特性之外，它还有一个鲜明的特点。以这种模式启动的activity都必须单独的放在一个任务栈中。因此，当用户之后多次启动这个activity都不会在创建这个activity的实例，除非这个任务栈已经被销毁。

>关于taskAffinity的使用效果说明：如果taskAffinity和singleTask配对使用，它是具有该模式的activity的目前任务栈名字，待启动的activity会运行在名字和taskAffinity相同的任务栈中；当taskAffinity和allowTaskReparenting结合使用的时候，假设在应用a中启动了应用b的activityC，回到桌面点击b的应用图标，这时候系统打开的是已经被启动的activityC，而不是应用b的主activity，即c从a的任务栈转移到了b的任务栈里面去。


>这篇博文参考了任玉刚编写的《android开发艺术探索》以及android官方开发文档。更多内容可以参考以下链接
>
>[Tasks and Back Stack](http://developer.android.com/guide/components/tasks-and-back-stack.html#q=viewgroup)
>
>[activity标签](http://developer.android.com/guide/topics/manifest/activity-element.html#aff)