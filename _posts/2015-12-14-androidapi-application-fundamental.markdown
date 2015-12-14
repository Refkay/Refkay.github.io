---
layout:post
author:"Lemon Yang"
date:2015-12-14 15:00:00
title:"appication fundamentals"
tags:
	-api
	-android
---

>本文译自android.com中Develop－>API Guides->Application Fundamentals一节。

---

安卓应用使用java语言编写。安卓的开发工具将你的代码（包括所有的数据和资源文件）编译成一个apk，这是一个后缀名为.apk压缩包。一个apk文件包括了一个安卓应用的所有内容，并且安卓系统设备使用这种apk文件安装应用程序。

一旦应用被安装到设备上，每一个应用程序都会运行在自己独有的安全沙箱中：

	1.安卓操作系统是一个基于Linux的多用户系统，每一个应用都是一个独立的进程。
	2.默认的情况下，系统会为每个应用分配一个独立的用户id（这个id仅被系统持有，对程序自身是透明的）。系统为应用中的文件设置访问权限因此只有相对应的用户id才能够访问应用的文件。
	3.每一个进程都会有自己独立的虚拟机，所有每一个应用都是独立运行，与其他应用之间是隔离的。
	4.默认的情况下，每一个应用都运行在自己独立的进程中。当应用的组件被启动的时候，安卓会开启一个进程，并且当当进程不再被使用或者系统需要为其他应用分配足够的内存的时候，就会关闭这个线程。

通过这样的方式，安卓系统实现了最小权限的原则。在默认的情况下，每一个应用都仅有完成自己工作的权限。通过这样的方法确保一个非常安全大体系，确保每个应用在没有赋予权限的情况下不能访问系统的其他部分。

尽管这样，安卓应用仍然有很多的方法与其他应用共享数据以及访问系统的服务：

	1.两个应用之间能够共享同一个用户id，因此他们之间能够互相访问各自的文件。为了节约系统的资源，拥有共同的用户ID的应用还可以运行在同一个进程之中并且使用同一个虚拟机（这种情况下应用之间 必须使用相同的签名证书）
	2.每一个应用可以请求访问设备数据的权限，包括用户的通讯录、短信、数据存储系统、相机、蓝牙以及其他。应用的权限需要在安装的时候由用户确认。
	
---

## 应用组件（App Components）
应用组件是安卓应用的基本要素。每一个组件都是系统进入程序的地方。并不是所有的组件都是用户进入口，有的组件依赖于其他组件，有的组件独立存在并起着特别的作用，每一个组件都是帮助构建的应用综合功能的独有部分。

有四种不同类型的组件。每一个组件都有着不同的作用和目的并且有自己生命周期。

### Activities

每一个activity代表着一个拥有用户界面的屏幕。比如，一个邮件的程序可以有一个activity展示一系列的新邮件，另一个activity用于写邮件，还有一个actvity用于读邮件。尽管这些activity一起构成了一个邮件应用的使用，但是每个activity之间是相互独立的。此外，不同的程序也可以启动其他应用的activity（只要目的应用允许）。比如，一个相机应用可以在邮件的应用中被启动。

一个activity是作为Activity的子类实现。

### Services

服务是为了实现耗时操作或者远程操作而运行在后台的组件。服务并没有提供用户的界面。比如，服务可以用于在后台播放音乐当用户正在其他应用界面的时候，或者在不阻塞用户界面的情况下获取网络数据。activity可以启动一个服务，或者绑定一个服务并与之进行通讯。

一个服务作为Services的子类实现。

### Content providers

content provider用于管理共享的应用数据。用户可以将数据存储在文件系统中、sqllite数据库中、网络上或者任何应用程序可以访问的持久化存储系统。通过content provider可以查询甚至修改这些数据（如果content provider允许的话）。比如，安卓系统提供了content provider管理用户的联系人信息。因此具备合适权限的应用都可以查询部分的content provider对某个人的信息进行读写。

content provider对于读写应用内部不共享的数据也一样的有效。比如便签应用使用content provider保存便签。

content provider作为的Content providers子类实现，并且必须实现一套标准的api方法才能确保与其他 应用进行数据的交换。

### Broadcast receivers

broadcast receiver是一个用于响应系统广播的组件。很多广播来源于系统。比图通知屏幕方向转变的广播、截图的广播。应用程序也可以发送广播，比如让其他应用程序知道一些文件已经下载到设备上并且可以使用。尽管broadcast receiver并没有用户界面，但是他们可以发出一个状态栏通知告知用户广播事件的发生。通常情况下，broadcast receiver只是扮演着其他组件的大门的角色，只是用于处理轻量级的工作。比如，它可以初始化一个服务用于完成基于该事件的工作。

broadcast receiver作为Broadcast receivers的子类实现，并且每一个广播传播一个intent对象。

---

作为安卓系统设计的独特的一面，一个应用可以启动另一个应用的组件。比如，如果你想用设备的相机拍一张照片，很可能的情况是，有另一个应用实现了（拍照）这个功能，而你的应用可以直接使用它，而不是在你的应用中开发一个新的activity用于拍照。你并不需要合并或者连接到相机应用的代码。你可以简单的启动相机里面的activity去完成拍照的动作。并且拍完的时候，照片会返回到你的应用并且你可以使用它。对于用户而言，这就像是相机应用是你应用中的一部分。

当系统启动一个组件的时候，系统会为应用启动一个进程（进程还没有启动的话）并实例化组件所需要的类。比如，如果你的应用启动了相机应用里面的activity去拍摄照片，那个被启动的activity会运行在属于相机应用的进程，而并不是你的应用的进程。因此，不像大多数系统的应用，安卓的应用并没有一个单一的入口。

因为系统将每个应用运行在独立的带有文件权限的进程中，你的应用不可以直接激活其他应用的组件。然而，系统本身可以。因此，为了激活其他应用的组件，你必须发送一个指明了你要启动的组件的意图信息给系统。并由系统为你激活相应的组件。

### Activating Components
有三种组件**（activity、service、broadcast receiver）**是被一种称之为意图**（intent）**的异步消息激活的。意图在运行期间绑定组件（你可以把这些意图对象理解成是那些等待着命令的送信人），不管这些组件是属于你的应用还是其他的。

一个意图是由意图对象产生的，它定义了一个激活明确的组件或者是某一类组件的消息——意图可以是显式的也可以是隐式的。

* 对于activity和service，一个意图定义了一个执行的动作（比如，显示或者是发送一些东西），也有可能指定了需要展示的数据的uri（以及是其他启动组件需要知道的东西）。比如，一个意图可以传达一个展示图片或者打开一个网页的需求。在某些情况下，你可以启动一个activity接收一个结果；也有的情况，activity会通过intent返回结果（比如，你可以分发一个意图让用户选择一个联系人并且让它返回给你——返回的intent包含了指向选择的联系人的uri）

* 对于broadcast receiver，意图只是简单定义了需要被广播的通知（）

* 至于content provider并不是由意图来激活的。一个来自于ContentResolver的需求指向某个content provider的时候，它才会被激活。

	>The content resolver handles all direct transactions with the content provider so that the component that's performing transactions with the provider doesn't need to and instead calls methods on theContentResolverobject. This leaves a layer of abstraction between the content provider and the component requesting information (for security).

有各种不同的方法用于激活各种类型的组件：

* 你可以启动一个activity通过传递一个意图到方法startActivity()或者是startActivityForResult()（当你希望启动的activity能够返回一个结果的时候）

* 你可以开启一个service通过传递一个意图到方法startService()。或者你可以绑定一个service通过 传递意图到方法bindService()。

* 你可以实例化一个广播通过传递一个意图到这些方法：sendBroadcast(),sendOrderedBroadcast(), orsendStickyBroadcast().

* 你可以在一个ContentResolver对象调用query()方法查询一个content provider。


### The Manifest File

在安卓系统启动一个应用组件之前，系统必须可以通过阅读应用的AndroidManifest.xml配置文件明确组件的存在。你的应用必须在这个文件里面声明所有的组件，并且这个文件必须在整改应用目录结构的根目录。

这个配置文件除了声明组件之外还做了一系列的事情，比如：

* 指出应用所需要的全部权限，比如访问网络、读取用户联系人信息的权限；

* 声明应用最低的api级别，应用基于哪一个级别的api开发；

* 声明应用所需要的硬件和软件参数，比如相机、蓝牙、多点触控屏幕；

* 应用需要连接的其他的api库（除了安卓基本框架库以外的），比如谷歌地图库；

* 更多其他的内容


### Declaring components

配置文件的主要任务就是告知系统应用的组件信息，例如：

>android:label="@string/example_label"...>


在这个元素节点中，**android:icon**属性指向了资源文件的一个用于指示应用程序的图标

在这个元素节点中，**android:name**这个属性指明了这个activity的全程类名（包括包名的），**android:label**属性指明了一个用户可见的activity标签的字符串。

你必须通过这样的方法声明所有的组件：

	elements for services
	elements for activities
	elements for broadcast receivers
	elements for content providers

**Activities, services, and content providers，这些你在代码中实现但是没有在配置文件中声明对于系统是不可见的，并且是不可以使用运行的。然而，广播接收者既可以在配置文件中声明，也可以在代码中动态生成并且通过调用registerReceiver().的方法向系统注册这个接收器**。

### Declaring component capabilities

正如我们在Activating Components一节中所讨论的，你可以使用意图去启动**activity、service、broadcast receiver**。你可以在intent中显式的指明目的组件（使用组件的类名）完成这些任务，然而，intent的真正的力量却在于它的隐式使用。一个隐式的intent可以简单的描述需要响应的action（你也可以选择基于哪种data类型响应），同时系统会在设备中寻找可以响应intent中描述的action的组件并启动它。如果有多个组件符合描述条件的话，用户可以选择使用其中一个。

系统判别可以响应intent的组件的方法是通过将设备中安装的应用的应用配置文件中所提供的intent filters与接收到的intent对象进行匹配比较。


当你在你的应用文件中声明一个activity的时候，你可以选择给activity添加intent filters表明它可以响应的来自于其他应用的intent。你可以给你的组件声明一个intent filter通过添加一个元素作为组件元素的字节点。例如：


然后，当其他应用生成了一个带有**ACTION_SEND**这个action的intent并将其传递到方法**startActivity()**的时候，系统就会启动你的activity，因此用户可以使用它起草并发送邮件。

### Declaring app requirements

各种各样的硬件设备使用安卓系统，而且并非所有的硬件设备提供相同的硬件参数和能力。为了避免你的应用被安装到不符合你的应用的硬件需求的设备上面，通过在应用配置文件中声明你的应用所支持的设备类型是非常重要的。系统并不会读取这些声明内容，但是一些外部服务（比如Google play）会读取这些声明为了给用户提供合适的应用当他们在自己的设备上搜索应用的时候。

比如，如果你的应用需要相机并且使用安卓2.1的系统，你应该在配置文件中这样声明：

>android:required="true"/>


这样子，所有没有相机或者是系统低于2.1的设备都不会从Google play上安装到你的应用。

然而，你可以声明你的应用会使用到相机，但并不是一定需要它。在这种情况下，你的应用应该设置**required**属性为false 并在运行的时候检测设备是否有相机并屏蔽相关的相机功能。