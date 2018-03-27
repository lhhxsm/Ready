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
	Handler通过发送和处理Message和Runnable对象来关联相对应线程的MessageQueue。  
	1.可以让相对应的Message和Runnable来在未来的某个时间点进行相应处理。  
	2.让自己相应处理的耗时线程操作放在子线程，让更新UI的操作放在主线程。
- Handler的使用  
	1.post(runnable)  
	2.sendMessage(message)