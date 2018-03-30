## Android的四种启动模式（launchMode） ##
1. standard： 默认启动模式，每次激活 Activity 的时候都会创建一个新的 Activity 实例，并放入任务栈中。添加A->添加B->添加B->退出B->退出B->退出A
2. singleTop：如果在任务的栈顶正好存有该 Activity 的实例，则会通过调用 onNewIntent() 方法进行重用，否则就会同 standard 模式一样，创建新的实例并放入栈顶。（ 当且仅当启动的 Activity 和上一个 Activity 一致的时候才会通过调用 onNewIntent() 方法重用 Activity ）   添加A->添加B->添加A 此时堆栈内是ABA(退出A->退出B->退出A)、添加A->添加B->添加B 此时堆栈内是AB（退出B->退出A）  如资讯阅读类APP的内容界面
3. singleTask： 只要栈中已经存在了该 Activity 的实例，就会直接调用 onNewIntent() 方法来实现重用实例。重用时，直接让该 Activity 的实例回到栈顶，并且移除之前它上面的所有 Activity 实例。添加A->B->C->D->B 此时栈内是AB（退出B->退出A）如浏览器主页或者大部分APP的主页
4. singleInstance： 在一个新栈中创建该 Activity 的实例，并让多个应用共享该栈中的该 Activity 实例。一旦该模式的 Activity 实例已经存在于某个栈中，任何应用再激活该 Activity 时都会重用该栈中的实例，是的，依然是调用 onNewIntent() 方法。 singleInstance 不要用于中间页面。Android系统的来电页面
5. 通过设置 Intent.setFlags(int flags) 来设置启动的 Activity 的启动模式。通过代码设置的Activity的启动模式，优先级高于清单文件设置的。
6. 可能遇到的问题：startActivityForResult启动一个Activity，还没有开始跳转就直接执行了onActivityResult（）、生命周期onResume（）被无故调用？原因：只要是不和原来的Activity在同一个Task就会产生onActivityResult（）被立即执行的情况，onActivityResult（）被执行，Activity会重新获取焦点，导致onResume（）被无故调用。

----------

## Activity的生命周期 ##
* Activity的4种状态：running/paused/stopped/killed
* 启动 Activiy：onCreate => onStart() => onResume(), Activity 进入运行状态.
* Activity 退居后台 ( Home 或启动新 Activity ): onPause() => onStop().
* Activity 返回前台: onRestart() => onStart() => onResume().
* 退出当前Activity:onPause() => onStop() => onDestroy().
* Activity 后台期间内存不足情况下当再次启动会重新执行启动流程。
* 锁屏: onPause() => onStop().
* 解锁: onStart() => onResume().
![](https://raw.githubusercontent.com/lhhxsm/Ready/master/picture/%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)
	1. onCreate() ：创建
	2. onStart()：Activity由不可见变为可见 
	3. onResume()：Activity准备好和用户进行交互时调用，此时的活动一定位于返回栈的栈顶，并且处于运行状态。
	4. onPause()：执行速度一定要快快，不然会影响到新的栈顶Activity的使用。
	5. onStop()：Activity完全不可见时调用。启动一个新的Activity是对话框时，onPause()方法会被调用,而onStop()方法不会被执行。
	6. onDestroy()：销毁
	7. onRestart()：由停止状态变成运行状态之前调用，重新启动。
* 可见生存期：onStart()和onStop()之间。初始化加载--释放资源
* 前台生存期：onResume()和onPause()之间。和用户进行交互
* 完整生存期：onCreate()和onDestroy()之间。

问题1.
假设项目中有这样的需求，当指定的 Activity 在用户可见后才进行广播的注册，在用户不可见后对广播进行注销，那应该在哪两个回调中做这个处理呢？  
答案1：onStart()和onStop().  
问题2：
如果有一些数据在 Activity 跳转时（或者离开时）要保存到数据库，那么你认为是在 onPause() 好还是在 onStop() 执行这个操作好呢？  
答案2：onPause(). 难以保证每次运行都能正常运行到 onStop()方法，比如还没运行到 onStop() 系统就被回收了。  

* Android进程优先级  前台、可见、服务、后台、空
* Android任务栈  

----------

scheme跳转协议
--

Android种是scheme是一种页面内跳转协议， 服务器定制化页面跳转、通知栏消息定制化跳转、H5页面跳转等。

Fragment
--
- FragmentPagerAdapter与FragmentStatePagerAdapter区别： 后者每次切换有回收内存，适用于页面较多的情况，更好内存。前者在页面切换是只是Fragment与Activity分离，适用于页面较少的情况
- Fragment的生命周期
![](https://raw.githubusercontent.com/lhhxsm/Ready/master/picture/Fragment%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)
- Fragment通信  
	1. 在Fragment中调用Activity中的方法 getActivity
	2. 在Activity中调用Fragment中的方法 接口回调
	3. 在Fragment中调用Fragment中的方法 findFragmentById
- FragmentManager replace(替换) add(增加) remove(移除)

----------

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
1. 本质是一个封装了线程池和Handler的异步框架。  
2. 机制原理：本质是一个静态线程池，AsyncTask派生的子类实现不同的异步任务，这些异步任务都是提交到静态线程池中执行的。工作线程doInBackgroud()方法执行异步任务。AsyncTask内部的InternalHandler响应这些消息，并调用相关的函数回调。
3. 注意事项：  
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
- 优先级高于Service，是继承并处理异步请求的一个类，内部有一个工作线程来处理耗时操作。当任务执行完成之后，IntentService会自动停止，不需要手动控制。可以启动多次，每次只执行一个工作线程。  
	1. 它本质是一种特殊的Service,继承自Service并且本身就是一个抽象类。  
	2. 它内部是由HandlerThread和Handler实现异步操作。
- 使用方法：实现onHandlerIntent和构造方法，onHandlerIntent为异步方法，可以执行耗时操作。

----------
## View绘制机制 ##
- View树的绘制流程：measure（测量）->layout（布局）->draw（绘制）

- Measure过程
![](https://raw.githubusercontent.com/lhhxsm/Ready/master/picture/Measure%E8%BF%87%E7%A8%8B.png)  
	1.ViewGroup.LayoutParams指定视图的宽和高  
	2.MeasureSpec（测量规格）：  
		a. MeasureSpec.EXACTLY //确定模式，父View希望子View的大小是确定的，由specSize决定；  
		b. MeasureSpec.AT_MOST //最多模式，父View希望子View的大小最多是specSize指定的值；  
		c. MeasureSpec.UNSPECIFIED //未指定模式，父View完全依据子View的设计值来决定；   
		d. View的measure方法是final的，不允许重载，View子类只能重载onMeasure来完成自己的测量逻辑。  
		e. View的布局大小由父View和子View共同决定，使用View的getMeasuredWidth()和getMeasuredHeight()方法来获取View测量的宽高，必须保证这两个方法在onMeasure流程之后被调用才能返回有效值。  
	  
- Layout过程  
	1.View.layout方法可被重载,ViewGroup.layout为final的不可重载，ViewGroup.onLayout为abstract的，子类必须重载实现自己的位置逻辑。  
	2.measure操作完成之后得到每个经过测量View的measuredWidth和measuredHeight。layout操作完成之后对每个View位置分配left、top、right、bottom，这些值是相对于父View来说的。  
	3.使用View的getWidth()和getHeight()方法来获取View测量的宽高，必须保证这两个方法在onLayout流程之后被调用才能返回有效值。
- Draw过程
	1. 如果该View是一个ViewGroup，则需要递归绘制其所包含的所有子View。
	2. View默认不会绘制任何内容，真正的绘制都需要自己在子类中实现。
	3. View的绘制是借助onDraw方法传入的Canvas类来进行的。
	4. 区分View动画和ViewGroup布局动画，前者指的是View自身的动画，可以通过setAnimation添加，后者是专门针对ViewGroup显示内部子视图时设置的动画，可以在xml布局文件中对ViewGroup设置layoutAnimation属性（譬如对LinearLayout设置子View在显示时出现逐行、随机、下等显示等不同动画效果）
	5. 在获取画布剪切区（每个View的draw中传入的Canvas）时会自动处理掉padding，子View获取Canvas不用关注这些逻辑，只用关心如何绘制即可。
	6. 默认情况下子View的ViewGroup.drawChild绘制顺序和子View被添加的顺序一致，但是你也可以重载ViewGroup.getChildDrawingOrder()方法提供不同顺序。
	7. invalidate()和requestLayout()区别

----------
## 事件分发机制 ##
1. 产生背景：View是树形结构，View可能会重叠在一起，点击一个地方会有多个View响应。
2. 分发方法  
	- dispatchTouchEvent 进行事件的分发。如果事件能够传递给当前View,那么此方法一定会被调用。
	- onInterceptTouchEvent 用来判断是否拦截某个事件。如果当前View拦截了某个事件，那么同一个事件序列当中，此方法不会被再次调用，返回结果表示是否拦截当前事件。
	- onTouchEvent 用来处理点击事件，返回结果表示是否消耗当前事件，如果不消耗，则在同一个事件序列中，当前View无法再次接收到事件。
3. 事件传递顺序：Activity–>Window–>View
4. 事件的分发流程  
	Activity->PhoneWindow->DecorView-ViewGroup(View树最底部的View)

----------
## ListView ##
1. recycleBin机制
2. ListView的优化  	  
	a. convertView重用机制  
	b. ViewHolder机制  
	c. 三级缓冲/滑动监听事件

----------
## 动画机制 ##
1. 补间动画：平移（Translate）、旋转（Rotate）、缩放（Scale）、透明度（Alpha）、组合（set）。平移之后焦点还在原来的位置，控件属性不变，优点是相对于
2. 帧动画，制作简单，效果单一，逐帧播放需要很多图片，占用控件较大
3. 属性动画（Android3.0/API 11 才出现的，之前的版本需要使用开源的项目nineoldandroids.jar），优点是易定制，效果强。

----------
## 自定义View ##
1. 自定义View的几种方式  
	a.对原有的View进行扩展   
	b.多个View的组合  
	c.重写View  
