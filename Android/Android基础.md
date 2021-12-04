# 一.Activity

### 1.onSaveInstanceState()调用时机？

答：(1)、当用户按下HOME键时。

　　(2)、长按HOME键，选择运行其他的程序时。

　　(3)、按下电源按键（关闭屏幕显示）时。

　　(4)、从activity A中启动一个新的activity时。

　　(5)、屏幕方向切换时，例如从竖屏切换到横屏时。

　　在屏幕切换之前，系统会销毁activity A，在屏幕切换之后系统又会自动地创建activity A，所以onSaveInstanceState()一定会被执行，且也一定会执行onRestoreInstanceState()。

　　总而言之，onSaveInstanceState()的调用遵循一个重要原则，即当系统存在“未经你许可”时销毁了我们的activity的 可能时，则onSaveInstanceState()会被系统调用，这是系统的责任，因为它必须要提供一个机会让你保存你的数据（当然你不保存那就随便 你了）。如果调用，调用将发生在onPause()或onStop()方法之前。（虽然测试时发现多数在onPause()前）

### 2.onRestoreInstanceState()调用时机？

答：onRestoreInstanceState只有在activity被系统回收，重新创建activity的情况下才会被调用。



### 3.onCreate()里也有Bundle参数，可以用来恢复数据，它和onRestoreInstanceState有什么区别？

答：因为onSaveInstanceState 不一定会被调用，所以onCreate()里的Bundle参数可能为空，如果使用onCreate()来恢复数据，一定要做非空判断。

而onRestoreInstanceState的Bundle参数一定不会是空值，因为它只有在上次activity被回收了才会调用。

而且onRestoreInstanceState是在onStart()之后被调用的。有时候我们需要onCreate()中做的一些初始化完成之后再恢复数据，用onRestoreInstanceState会比较方便。



### 4.Activity的启动模式

答：

##### 1.standard：标准启动模式（默认启动模式）

 每次启动Activity都会创建一个新的实例。Activity会进入启动它的Activity的任务栈中。

##### 2.singleTop：栈顶复用模式(顶单例模式)

如果要启动的Activity实例<font color=#ff0000>已经位于当前任务栈顶</font>，就会重用栈顶实例，不会重新创建。并且<font color=#ff0000>回调该实例的onNewIntent()方法</font>，否则走新建的流程。

如果将要被启动的<font color=#ff0000>目标Activity没有位于任务栈栈顶</font>的话，此时系统会重新目标Activity，并将创建的新的Activity实例添加到任务栈中（此时与standard加载模式相同）。

##### 3.singleTask:栈内复用模式(内单例模式)

这种模式启动的Activity**只会存在相应的Activity的taskAffinity任务栈中**，系统中只会存在一个实例，已存在的实例被再次启动时，会重新唤起该实例，并清理当前Task任务栈该实例之上的所有Activity，同时<font color=#ff0000>回调该实例的onNewIntent()</font>方法。

##### 4.singleInstance:全局单例模式(独享一个Task)

这种模式启动的Activity独自占用一个Task任务栈，系统中只会存在一个实例，已存在的实例被再次启动时，只会唤起原实例，并回调onNewIntent()方法。

##### singleTask适合作为程序入口点。

例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。

##### singleInstance应用场景：

闹铃的响铃界面。 你以前设置了一个闹铃：上午6点。在上午5点58分，你启动了闹铃设置界面，并按 Home 键回桌面；在上午5点59分时，你在微信和朋友聊天；在6点时，闹铃响了，并且弹出了一个对话框形式的 Activity(名为 AlarmAlertActivity) 提示你到6点了(这个 Activity 就是以 SingleInstance 加载模式打开的)，你按返回键，回到的是微信的聊天界面，这是因为 AlarmAlertActivity 所在的 Task 的栈只有他一个元素， 因此退出之后这个 Task 的栈空了。如果是以 SingleTask 打开 AlarmAlertActivity，那么当闹铃响了的时候，按返回键应该进入闹铃设置界面。



### 5.Activity A跳转Activity B，再按返回键，生命周期执行的顺序?

### 6.分别说一下横竖屏切换,按home键,按返回键,锁屏与解锁屏幕,生命周期执行的顺序?

### 7.跳转透明Activity界面,启动一个 Theme 为 Dialog 的 Activity，弹出Dialog时Activity的生命周期?



### 8.Activity之间传递数据的方式Intent是否有大小限制，如果传递的数据量偏大，有哪些方案？

答：通过intent在页面间传递数据是有大小限制的.

通过源码查看 The Binder transaction buffer has a limited fixed size, currently 1Mb, which
is shared by all transactions in progress for the process.  Consequently this
exception can be thrown when there are many transactions in progress even when
most of the individual transactions are of moderate size. 

解决方案：

一、持久化数据:写入临时文件或者数据库，通过FileProvider将该文件或者数据库通过Uri发送至目标。一般适用于不同进程，比如分离进程的UI和后台服务，或不同的App之间。之所以采用FileProvider是因为7.0以后，对分享本App文件存在着严格的权限检查。

二、通过设置静态类中的静态变量进行数据交换。一般适用于同一进程内，这样本质上数据在内存中只存在一份，通过静态类进行传递。需要注意的是进行数据校对，以防多线程操作导致的数据显示混乱。

三、使用EventBus或者其他消息总线的工具。

### 9.Activity任务栈是什么？

答:任务是用户在执行某项工作时与之互动的一系列 Activity 的集合。这些 Activity 按照每个 Activity 打开的顺序排列在一个返回堆栈中。

Android 管理任务和返回堆栈的方式是将所有接连启动的 Activity 放到同一任务和一个“后进先出”堆栈中。



# 二.Service

### 1.service 的生命周期，两种启动方式的区别?

答:![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7631e3b7e46a46b1992b630671f2fd82~tplv-k3u1fbpfcp-watermark.awebp)

#### 启动方式：

StartService使用的是同一个Service，因此onStart()会执行多次，onCreate()只执行一次，onStartCommand()也会执行多次。使用bindService启动时，onCreate()与onBind()都只会调用一次。

使用startService启动时是单独开一个服务，与Activity没有任何关系，而bindService方式启动时，Service会和Activity进行绑定，当对应的activity销毁时，对应的Service也会销毁。



### 2.Activity与Service的通信方式？

答：1.不论是start还是bind，都可以在开始时通过intent携带一定量数据。

​        2.通过Broadcast，Activity发送广播，Service接收；或者Service发送广播，Activity接收。

​        3.通过bind方式启动时，可以通过实现ServiceConnection来完成Activity和Service的交互。

### 3.IntentService是什么,IntentService原理，应用场景及其与Service的区别？

答：IntentService是Service 的子类，默认给我们开启了一个工作线程执行耗时任务，并且执行完任务后自 动停止服务。扩展IntentService比较简单，提供一个构造方法和实现onHandleIntent 方法就可了，不用重写父类的其他方法。但是如果要绑定服务的话，还是要重写onBind 返回一个IBinder 的。

**使用Service 可以同时执行多个请求，而使用IntentService 只能同时执行一个请求。**

IntentService与service的区别：

1：Service服务是长期运行在后台；
 2：它不是单独的进程，因为它和应用程序在同一个进程；
 3：也不是单独的线程，它跟线程没有任何关系，所以不能进行耗时操作；
 4：如果直接把耗时操作放在Service中的onStartCommand()中，可能发生ANR，如果有耗时操作，就必须开启一个单独的线程来处理；

为了解决这样的问题，就引入了IntentService；

 1：IntentService启动方式和Service一样，都是startService()；
 2：继承于Service，包含Service所有特性，包括生命周期，是处理异步请求的一个类；
 3：一般自定义一个InitializeService继承Service，然后复写onHandleIntent()方法，在这个方法中初始化这些第           三方的，来执行耗时操作；
 4：可以启动多次IntentService，每一个耗时操作以工作队列在onHandleIntent()方法中执行，执行完第一个再去执行第二个，以此类推；
 5：所有的请求都在单线程中，不会阻塞主线程，同一个时间只处理同一个请求；
 6：不需要像在Service中一样，手动开启线程，任务执行完成后不需要手动调用stopSelf()方法来停止服务，系统会自动关闭服务；













