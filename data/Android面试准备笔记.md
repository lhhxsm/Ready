## BroadcastReceiver 广播 ##
- 广播的定义
- 广播的使用场景：  
	1.同一APP具有多个进程的不同组件直接的消息通信  
	2.不同APP之间的组件之间的消息通信
- 广播的种类  
	1.普通广播sendBroadcast  
	2.有序广播sendOrderBroadcast  
	3.本地广播（只在自身APP内传播）Local Broadcast
- 实现广播-receiver  
	1.静态注册：注册完成就一直运行（注册的Activity销毁了，广播任然能收到广播）  
	2.动态注册：跟随Activity的生命周期  
	3.区别：a.动态注册是在代码中注册，静态注册是在AndroidManifest中注册。b.动态注册受Activity的生命周期影响。
- 广播的内部实现机制  
	1.自定义广播接受者BroadcastReceiver，并复写onReceive()方法  
	2.通过Binder机制向AMS（Activity Manger Service）注册  
	3.广播发送者通过Binder机制向AMS发送广播  
	4.AMS查找符合相应条件（IntentFilter/Permission等）的BroadcastReceiver，将广播发送到BroadcastReceiver相应的消息队列中  
	5.消息循环执行拿到此广播，回调BroadcastReceiver中的onReceive()方法  
**- AMS（Activity Manager Service）负责启动四大组件，切换，调度，以及应用程序的管理和调度。**  
**- Binder机制 C/S架构**  
- LocalBroadcastManager  
	1.只在APP内传播，因此不必担心泄露隐私数据  
	2.其他APP无法对自己的APP发送广播，不可能接收到非自身应用发送的广播，因此不必担心有安全漏洞可以利用  
	3.比系统的全局广播更加高效 内部是通过Handler实现的  

----------
## Webview详解 ##

- 产生漏洞的原因：程序没有正确限制使用WebView.addJavaScriptInterface方法，攻击者通过使用Java Reflection API利用该漏洞执行任意Java对象的方法
- WebView在布局文件中的使用：WebView写在其他容器中时，导致内存泄露。需要先在容器中销毁，在销毁WebView
- jsbridge 调用远端的Web等
- 后台耗电，WebView使用之后一定要销毁
- WebView硬件加速导致页面渲染问题，容易出现界面单锁，白块，解决办法关闭硬件加速
- 解决内存泄露：  
	1.独立进程  
	2.动态添加WebView

----------
## Binder ##
- Linux内核的基础知识  
	1.进程隔离/虚拟地址空间  
	2.系统调用  
	3.Binder驱动
- Binder通信机制  
	1.Android使用的Linux内核拥有非常多的跨进程通信机制  
	2.性能  
	3.安全  
	4.Binder驱动，运行在内核里的程序
- 什么是Binder  
	1.Binder指的是一种通信机制  
	2.对于Server进程来说，Binder指的是Binder**本地对象**/对于Client来说，Binder指的是Binder**代理对象**  
	3.对于传输过程而言，Binder是可以进行跨进程传递的对象
- AIDL   
	

----------
## Handler ##
- 什么是Handler？  
	Handler通过发送和处理Message和Runnable对象来关联相对应线程的MessageQueue。是一种异步的机制。  
	1.可以让相对应的Message和Runnable来在未来的某个时间点进行相应处理。  
	2.让自己相应处理的耗时线程操作放在子线程，让更新UI的操作放在主线程。而子线程与主线程之间的通信就是靠Handler来完成的。
- Handler的使用  
	1.post(runnable)  
	2.sendMessage(message)
- Handler内部实现机制  
	1.Message：是在线程之间传递的消息。用于在不同线程之间的交换数据。  
	2.Handler：处理子线程和UI线程的消息对象Message。在子线程调用sendMessage发送消息对象，在handleMessage方法处理Message。  
	3.MessageQueue：消息队列，用于存放所有通过Handler发送过来的消息。消息会一直存放于消息队列当中，等待被处理。每个线程中只会有一个MessageQueue对象。MessageQueue底层数据结构是队列，而且这个队列只存放Message。  
	4.Looper:调用Looper的loop()方法后，进入到一个无限循环当中，然后每当MesssageQueue中存在一条消息，Looper就会将这条消息取出，并将它传递到Handler的handleMessage()方法中。每个线程只有一个Looper对象。
![](https://raw.githubusercontent.com/lhhxsm/Ready/master/picture/Handler%E6%9C%BA%E5%88%B6.png)
	
- Handler引起的内存泄漏   
	1.原因：静态内部类持有外部类的匿名引用，导致外部activity无法得到释放。  
	2.解决方法：handler内部持有外部的弱引用，并把handler改成静态内部类，在activity的onDestroy()中调用handler的removeCallback()方法。

----------
## AsyncTask机制   ##
1.本质是一个封装了线程池和Handler的异步框架。  
2.注意事项：  
	a.内存泄漏：静态内部类持有外部类的匿名引用，导致外部对象无法得到释放，解决方法是让内部持有外部的弱引用。  
	b.生命周期：在onDestroy中进行回收，调用cancel方法。
	c.结果丢失：当内存不足时，当前的Activity被回收，由于AsyncTask持有的是回收之前Activity的引用，导致AsyncTask更新的结果对象为一个无效的Activity的引用，这就是结果丢失。
	d.串行或并行

----------
## HandlerThread机制 ##
- 产生背景：开启子线程进行耗时操作，多次创建和销毁子线程是很耗费资源的。
- HandlerThread是什么？本质：Handler+Thread+Looper  
	1.HandlerThread本质是一个线程类，继承Thread。  
	2.HandlerThread有自己内部的Looper对象，可以进行Looper循环。  
	3.通过获取HandlerThread的Looper对象传递给Handler对象，可以在handlerMessage方法中执行异步任务。  
	4.优点是不会有堵塞，减少对性能的消耗，缺点是不能进行多任务的处理，需要等待进行处理，处理效率较低。  
	5.与线程池注重并发不同，HandlerThread是一个串行队列，HandlerThread背后只有一个线程。

----------
## IntentService ##
- 优先级高于Service，是继承处理异步请求的一个类，内部有一个工作线程来处理耗时操作。当任务执行完成之后，IntentService会自动停止，不需要手动控制。可以启动多次，每次只执行一个工作线程。  
	a.它本质是一种特殊的Service,继承自Service并且本身就是一个抽象类。  
	b.它内部是由HandlerThread和Handler实现异步操作。
- 使用方法：实现onHandlerIntent和构造方法，onHandlerIntent为异步方法，可以执行耗时操作。

----------
## View绘制机制 ##
- View树的绘制流程：measure（测量）->layout（布局）->draw（绘制）
- Measure过程