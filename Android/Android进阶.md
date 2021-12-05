# 1.什么是ANR 如何避免它？

答：在Android上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应 用程序无响应（ANR：Application NotResponding）对话框。 用户可以选择让程序继续运行，但是，他们在使用你的 应用程序时，并不希望每次都要处理这个对话框。因此 ，在程序里对响应性能的设计很重要这样，这样系统就不会显 示ANR给用户。

不同的组件发生ANR的时间不一样，Activity是5秒，BroadCastReceiver是10秒，Service是20秒（均为前台）。

如果开发机器上出现问题，我们可以通过查看/data/anr/traces.txt即可，最新的ANR信息在最开始部分。

- 主线程被IO操作（从4.0之后网络IO不允许在主线程中）阻塞。
- 主线程中存在耗时的计算
- 主线程中错误的操作，比如Thread.wait或者Thread.sleep等 Android系统会监控程序的响应状况，一旦出现下面两种情况，则弹出ANR对话框
- 应用在5秒内未响应用户的输入事件（如按键或者触摸）
- BroadcastReceiver未在10秒内完成相关的处理
- Service在特定的时间内无法处理完成 20秒

##### 解决方案：

将所有耗时操作，比如访问网络，Socket通信，查询大 量SQL 语句，复杂逻辑计算等都放在子线程中去，然 后通过handler.sendMessage、runonUIThread、AsyncTask、RxJava等方式更新UI。无论如何都要确保用户界面的流畅 度。如果耗时操作需要让用户等待，那么可以在界面上显示度条。

# 2.android中进程的优先级？

##### 1. 前台进程：

即与用户正在交互的Activity或者Activity用到的Service等，如果系统内存不足时前台进程是最晚被杀死的

##### 2. 可见进程：

可以是处于暂停状态(onPause)的Activity或者绑定在其上的Service，即被用户可见，但由于失了焦点而不能与用户交互

##### 3. 服务进程：

其中运行着使用startService方法启动的Service，虽然不被用户可见，但是却是用户关心的，例如用户正在非音乐界面听的音乐或者正在非下载页面下载的文件等；当系统要空间运行，前两者进程才会被终止

##### 4. 后台进程：

其中运行着执行onStop方法而停止的程序，但是却不是用户当前关心的，例如后台挂着的QQ，这时的进程系统一旦没了有内存就首先被杀死

##### 5. 空进程：

不包含任何应用程序的进程，这样的进程系统是一般不会让他存在的。

# 3.Bunder传递对象为什么需要序列化？Serialzable和Parcelable的区别？

因为bundle传递数据时只支持基本数据类型，所以在传递对象时需要序列化转换成可存储或可传输的本质状态（字节流）。序列化后的对象可以在网络、IPC（比如启动另一个进程的Activity、Service和Reciver）之间进行传输，也可以存储到本地。

##### Serializable（Java自带）：

Serializable 是序列化的意思，表示将一个对象转换成存储或可传输的状态。序列化后的对象可以在网络上进传输，也可以存储到本地。

##### Parcelable（android专用）：

除了Serializable之外，使用Parcelable也可以实现相同的效果，不过不同于将对象进行序列化，Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这也就实现传递对象的功能了。

# 4.Context相关

1、Activity和Service以及Application的Context是不一样的,Activity继承自ContextThemeWraper.其他的继承自ContextWrapper。

2、每一个Activity和Service以及Application的Context是一个新的ContextImpl对象。

3、getApplication()用来获取Application实例的，但是这个方法只有在Activity和Service中才能调用的到。那也许在绝大多数情况下我们都是在Activity或者Servic中使用Application的，但是如果在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法，getApplicationContext()比getApplication()方法的作用域会更广一些，任何一个Context的实例，只要调用getApplicationContext()方法都可以拿到我们的Application对象。

4、创建对话框时不可以用Application的context，只能用Activity的context。

5、Context的数量等于Activity的个数 + Service的个数 +1，这个1为Application。



# 5.更新UI方式

1.Activity.runOnUiThread(Runnable)

2.View.post(Runnable)，View.postDelay(Runnable, long)（可以理解为在当前操作视图UI线程添加队列）

3.Handler

4.AsyncTask

5.Rxjava

6.LiveData



# 6.ContentProvider使用方法

进行跨进程通信，实现进程间的数据交互和共享。通过Context 中 getContentResolver() 获得实例，通过 Uri匹配进行数据的增删改查。ContentProvider使用表的形式来组织数据，无论数据的来源是什么，ConentProvider 都会认为是一种表，然后把数据组织成表格。



# 7.activity的startActivity和context的startActivity区别？

(1)、从Activity中启动新的Activity时可以直接mContext.startActivity(intent)就好

(2)、如果从其他Context中启动Activity则必须给intent设置Flag:

intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK) ; 
mContext.startActivity(intent);



# 8.Asset目录与res目录的区别？

assets：不会在 R 文件中生成相应标记，存放到这里的资源在打包时会打包到程序安装包中。（通过 AssetManager 类访问这些文件）

res：会在 R 文件中生成 id 标记，资源在打包时如果使用到则打包到安装包中，未用到不会打入安装包中。

res/anim：存放动画资源。

res/raw：和 asset 下文件一样，打包时直接打入程序安装包中（会映射到 R 文件中）。

# 9.Android怎么加速启动Activity？

onCreate() 中不执行耗时操作 把页面显示的 View 细分一下，放在 AsyncTask 里逐步显示，用 Handler 更好。这样用户的看到的就是有层次有步骤的一个个的 View 的展示，不会是先看到一个黑屏，然后一下显示所有 View。最好做成动画，效果更自然。

利用多线程的目的就是尽可能的减少 onCreate() 和 onReume() 的时间，使得用户能尽快看到页面，操作页面。

减少主线程阻塞时间。

提高 Adapter 和 AdapterView 的效率。

优化布局文件。

# 10.类的初始化顺序依次是？

（静态变量、静态代码块）>（变量、代码块）>构造方法

# 11.Bitmap 使用时候注意什么？

1、要选择合适的图片规格（bitmap类型）：

```
ALPHA_8   每个像素占用1byte内存        
ARGB_4444 每个像素占用2byte内存       
ARGB_8888 每个像素占用4byte内存（默认）      
RGB_565 每个像素占用2byte内存
```

2、降低采样率。BitmapFactory.Options 参数inSampleSize的使用，先把options.inJustDecodeBounds设为true，只是去读取图片的大小，在拿到图片的大小之后和要显示的大小做比较通过calculateInSampleSize()函数计算inSampleSize的具体值，得到值之后。options.inJustDecodeBounds设为false读图片资源。

3、复用内存。即，通过软引用(内存不够的时候才会回收掉)，复用内存块，不需要再重新给这个bitmap申请一块新的内存，避免了一次内存的分配和回收，从而改善了运行效率。

4、使用recycle()方法及时回收内存。

5、压缩图片。



# 12.OOM 是否可以try catch ？

只有在一种情况下，这样做是可行的：

在try语句中声明了很大的对象，导致OOM，并且可以确认OOM是由try语句中的对象声明导致的，那么在catch语句中，可以释放掉这些对象，解决OOM的问题，继续执行剩余语句。

但是这通常不是合适的做法。

Java中管理内存除了显式地catch OOM之外还有更多有效的方法：比如SoftReference, WeakReference, 硬盘缓存等。 在JVM用光内存之前，会多次触发GC，这些GC会降低程序运行的效率。 如果OOM的原因不是try语句中的对象（比如内存泄漏），那么在catch语句中会继续抛出OOM。



# 13.多进程场景遇见过么？为什么这么做，有什么优势？

1、在新的进程中，启动前台Service，播放音乐。 2、一个成熟的应用一定是多模块化的。首先多进程开发能为应用解决了OOM问题，因为Android对内存的限制是针对于进程的，所以，当我们需要加载大图之类的操作，可以在新的进程中去执行，避免主进程OOM。而且假如图片浏览进程打开了一个过大的图片，java heap 申请内存失败，该进程崩溃并不影响我主进程的使用。

# 14.Canvas.save()跟Canvas.restore()的调用时机？

save：用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。

restore：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。

save和restore要配对使用（restore可以比save少，但不能多），如果restore调用次数比save多，会引发Error。save和restore操作执行的时机不同，就能造成绘制的图形不同。

# 15.编译期注解跟运行时注解

运行期注解(RunTime)利用反射去获取信息还是比较损耗性能的，对应@Retention	（RetentionPolicy.RUNTIME）。

编译期(Compile time)注解，以及处理编译期注解的手段APT和Javapoet，对应@Retention(RetentionPolicy.CLASS)。 其中apt+javaPoet目前也是应用比较广泛，在一些大的开源库，如EventBus3.0+,页面路由 ARout、Dagger、Retrofit等均有使用的身影，注解不仅仅是通过反射一种方式来使用，也可以使用APT在编译期处理。

# 16.强引用置为null，会不会被回收？

不会立即释放对象占用的内存。 如果对象的引用被置为null，只是断开了当前线程栈帧中对该对象的引用关系，而 垃圾收集器是运行在后台的线程，只有当用户线程运行到安全点(safe point)或者安全区域才会扫描对象引用关系，扫描到对象没有被引用则会标记对象，这时候仍然不会立即释放该对象内存，因为有些对象是可恢复的（在 finalize方法中恢复引用 ）。只有确定了对象无法恢复引用的时候才会清除对象内存。

# 17.Bundle传递数据为什么需要序列化？

序列化，表示将一个对象转换成可存储或可传输的状态。序列化的原因基本三种情况：

1.永久性保存对象，保存对象的字节序列到本地文件中；

2.对象在网络中传递；

3.对象在IPC间传递。

# 18.自定义view效率高于xml定义吗？说明理由。

自定义view效率高于xml定义。

1、少了解析xml。

2.、自定义View 减少了ViewGroup与View之间的测量,包括父量子,子量自身,子在父中位置摆放,当子view变化时,父的某些属性都会跟着变化。

# 19.Handler、Thread和HandlerThread的差别

1、Handler：在android中负责发送和处理消息，通过它可以实现其他支线线程与主线程之间的消息通讯。

2、Thread：Java进程中执行运算的最小单位，亦即执行处理机调度的基本单位。某一进程中一路单独运行的程序。

3、HandlerThread：一个继承自Thread的类HandlerThread，Android中没有对Java中的Thread进行任何封装，而是提供了一个继承自Thread的类HandlerThread类，这个类对Java的Thread做了很多便利的封装。HandlerThread继承于Thread，所以它本质就是个Thread。与普通Thread的差别就在于，它在内部直接实现了Looper的实现，这是Handler消息机制必不可少的。有了自己的looper，可以让我们在自己的线程中分发和处理消息。如果不用HandlerThread的话，需要手动去调用Looper.prepare()和Looper.loop()这些方法。



# 20.MVP，MVVM，MVC解释和实践

##### MVC:

- 视图层(View) 对应于xml布局文件和java代码动态view部分
- 控制层(Controller) MVC中Android的控制层是由Activity来承担的，Activity本来主要是作为初始化页面，展示数据的操作，但是因为XML视图功能太弱，所以Activity既要负责视图的显示又要加入控制逻辑，承担的功能过多。
- 模型层(Model) 针对业务模型，建立数据结构和相关的类，它主要负责网络请求，数据库处理，I/O的操作。

##### 总结

具有一定的分层，model彻底解耦，controller和view并没有解耦 层与层之间的交互尽量使用回调或者去使用消息机制去完成，尽量避免直接持有 controller和view在android中无法做到彻底分离，但在代码逻辑层面一定要分清 业务逻辑被放置在model层，能够更好的复用和修改增加业务。

##### MVP

通过引入接口BaseView，让相应的视图组件如Activity，Fragment去实现BaseView，实现了视图层的独立，通过中间层Preseter实现了Model和View的完全解耦。MVP彻底解决了MVC中View和Controller傻傻分不清楚的问题，但是随着业务逻辑的增加，一个页面可能会非常复杂，UI的改变是非常多，会有非常多的case，这样就会造成View的接口会很庞大。

##### MVVM

MVP中我们说过随着业务逻辑的增加，UI的改变多的情况下，会有非常多的跟UI相关的case，这样就会造成View的接口会很庞大。而MVVM就解决了这个问题，通过双向绑定的机制，实现数据和UI内容，只要想改其中一方，另一方都能够及时更新的一种设计理念，这样就省去了很多在View层中写很多case的情况，只需要改变数据就行。

##### MVVM与DataBinding的关系？

MVVM是一种思想，DataBinding是谷歌推出的方便实现MVVM的工具。

看起来MVVM很好的解决了MVC和MVP的不足，但是由于数据和视图的双向绑定，导致出现问题时不太好定位来源，有可能数据问题导致，也有可能业务逻辑中对视图属性的修改导致。如果项目中打算用MVVM的话可以考虑使用官方的架构组件ViewModel、LiveData、DataBinding去实现MVVM。

##### 三者如何选择？

- 如果项目简单，没什么复杂性，未来改动也不大的话，那就不要用设计模式或者架构方法，只需要将每个模块封装好，方便调用即可，不要为了使用设计模式或架构方法而使用。
- 对于偏向展示型的app，绝大多数业务逻辑都在后端，app主要功能就是展示数据，交互等，建议使用mvvm。
- 对于工具类或者需要写很多业务逻辑app，使用mvp或者mvvm都可。

# 21.SharedPrefrences的apply和commit有什么区别？

1. apply没有返回值而commit返回boolean表明修改是否提交成功。
2. apply是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内容，后面有调用apply的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率。
3. apply方法不会提示任何失败的提示。 由于在一个进程中，sharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的。



# 22.单例实现线程的同步的要求？

1.单例类确保自己只有一个实例(构造函数私有:不被外部实例化,也不被继承)。

2.单例类必须自己创建自己的实例。

3.单例类必须为其他对象提供唯一的实例。

# 23.ContentProvider、ContentResolver、ContentObserver 之间的关系？

ContentProvider：管理数据，提供数据的增删改查操作，数据源可以是数据库、文件、XML、网络等，ContentProvider为这些数据的访问提供了统一的接口，可以用来做进程间数据共享。

ContentResolver：ContentResolver可以为不同URI操作不同的ContentProvider中的数据，外部进程可以通过ContentResolver与ContentProvider进行交互。

ContentObserver：观察ContentProvider中的数据变化，并将变化通知给外界。

# 24.Android中跨进程通讯的几种方式？

1：访问其他应用程序的Activity 如调用系统通话应用

```
Intent callIntent = new Intent(Intent.ACTION_CALL,Uri.parse("tel:12345678");
startActivity(callIntent);
```

2：Content Provider 如访问系统相册

3：广播（Broadcast） 如显示系统时间

4：AIDL服务



# 25.屏幕适配的处理技巧都有哪些?

##### 一、为什么要适配

为了保证用户获得一致的用户体验效果,使得某一元素在Android不同尺寸、不同分辨率的、不同系统的手机上具备相同的显示效果，能够保持界面上的效果一致,我们需要对各种手机屏幕进行适配！

- Android系统碎片化：基于Google原生系统，小米定制的MIUI、魅族定制的flyme、华为定制的EMUI等等；
- Android机型屏幕尺寸碎片化：5寸、5.5寸、6寸等等；
- Android屏幕分辨率碎片化：320x480、480x800、720x1280、1080x1920等。

##### 二、基本概念

- 像素（px）：像素就是手机屏幕的最小构成单元，px = 1像素点 一般情况下UI设计师的设计图会以px作为统一的计量单位。
- 分辨率：手机在横向、纵向上的像素点数总和 一般描述成 宽*高 ，即横向像素点个数 * 纵向像素点个数（如1080 x 1920），单位：px。
- 屏幕尺寸：手机对角线的物理尺寸。单位 英寸（inch），一英寸大约2.54cm 常见的尺寸有4.7寸、5寸、5.5寸、6寸。
- 屏幕像素密度（dpi）：每英寸的像素点数，例如每英寸内有160个像素点，则其像素密度为160dpi，单位：dpi（dots per inch）。
- 标准屏幕像素密度（mdpi）： 每英寸长度上还有160个像素点（160dpi），即称为标准屏幕像素密度（mdpi）。
- 密度无关像素（dp）：与终端上的实际物理像素点无关，可以保证在不同屏幕像素密度的设备上显示相同的效果，是安卓特有的长度单位，dp与px的转换：1dp = （dpi / 160 ） * 1px。
- 独立比例像素（sp）：字体大小专用单位 Android开发时用此单位设置文字大小，推荐使用12sp、14sp、18sp、22sp作为字体大小。

##### 三、适配方案

适配的最多的3个分辨率：1280*720,1920*1080,800*480。

解决方案：

对于Android的屏幕适配，我认为可以从以下4个方面来做：

1、布局组件适配

- 请务必使用密度无关像素 dp 或独立比例像素 sp 单位指定尺寸。
- 使用相对布局或线性布局，不要使用绝对布局
- 使用wrap_content、match_parent、权重
- 使用minWidth、minHeight、lines等属性

dimens使用：

不同的屏幕尺寸可以定义不同的数值，或者是不同的语言显示我们也可以定义不同的数值，因为翻译后的长度一般都不会跟中文的一致。此外，也可以使用百分比布局或者AndroidStudio2.2的新特性约束布局。

2、布局适配

使用限定符（屏幕密度限定符、尺寸限定符、最小宽度限定符、布局别名、屏幕方向限定符)根据屏幕的配置来加载相应的UI布局。

3、图片资源适配

使用自动拉伸图.9png图片格式使图片资源自适应屏幕尺寸。

普通图片和图标：

建议按照官方的密度类型进行切图即可，但一般我们只需xxhdpi或xxxhdpi的切图即可满足我们的需求；

4、代码适配：

在代码中使用Google提供的API对设备的屏幕宽度进行测量，然后按照需求进行设置。

5、接口配合：

本地加载图片前判断手机分辨率或像素密度，向服务器请求对应级别图片。



# 26.断点续传实现？

##### 基础知识：

- Http基础：在Http请求中，可以加入请求头Range，下载指定区间的文件数。
- RandomAccessFile：支持随机访问，可以从指定位置进行数据的读写。

有了这个基础以后，思路就清晰了：

- 通过HttpUrlConnection获取文件长度。
- 自己分配好线程进行制定区间的文件数据的下载。
- 获取到数据流以后，使用RandomAccessFile进行指定位置的读写。

在本地下载过程中要使用数据库实时存储到底存储到文件的哪个位置了，这样点击开始继续传递时，才能通过HTTP的GET请求中的setRequestProperty("Range","bytes=startIndex-endIndex");方法可以告诉服务器，数据从哪里开始，到哪里结束。同时在本地的文件写入时，RandomAccessFile的seek()方法也支持在文件中的任意位置进行写入操作。最后通过广播或事件总线机制将子线程的进度告诉Activity的进度条。关于断线续传的HTTP状态码是206，即HttpStatus.SC_PARTIAL_CONTENT。



# 27.讲讲 Android 的事件分发机制

![事件分发图](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/21/17237beb2491d7e5~tplv-t2oaga2asx-watermark.awebp)



判断是否需要拦截 —> 主要是根据 onInterceptTouchEvent 方法的返回值来决定是否拦截；

在 DOWN 事件中将 touch 事件分发给子 View —> 这一过程如果有子 View 捕获消费了 touch 事件，会对 mFirstTouchTarget 进行赋值；

最后一步，DOWN、MOVE、UP 事件都会根据 mFirstTouchTarget 是否为 null，决定是自己处理 touch 事件，还是再次分发给子 View。

DOWN 事件是事件序列的起点；决定后续事件由谁来消费处理；

mFirstTouchTarget 的作用：记录捕获消费 touch 事件的 View，是一个链表结构；

CANCEL 事件的触发场景：当父视图先不拦截，然后在 MOVE 事件中重新拦截，此时子 View 会接收到一个 CANCEL 事件。

如果一个事件最后所有的 View 都不处理的话，最终回到 Activity 的 onTouchEvent 方法里面来。



# 28.Android子线程中能不能更新UI？

at android.view.ViewRootImpl.checkThread(ViewRootImpl.java:7752 

在ViewRootImpl中，有一个方法 void checkThread() {

​        if (mThread != Thread.currentThread()) {

​            throw new CalledFromWrongThreadException(

​                    "Only the original thread that created a view hierarchy can touch its views.");

​        }

​    }

在此会有个检查线程的判断，只有创建视图原始线程才能接触其视图。并不是说，只有UI线程才能接触这个视图。你ViewRootImpl在哪个线程创建的，你后续的UI更新就需要在哪个线程执行，跟是不是UI线程毫无关系。

在这个前提之前，对于上述问题，有以下理解：

(1)因为ViewRootImp实例化是在onResume以后,所以在Activity的onResume之前，使用子线程更新UI，不会导致应用崩溃。

（2）如果在子线程调用dialog.show，是可以在dialog中正常的更新UI的，因为dialog调用show方法后，其ViewRootImpl是在子线程中创建的。如果此时更新UI，切换到UI线程会导致应用崩溃。



通过打印ViewParent的体系，会发现最底部是ViewRootImpl，所以当view触发了requesLayout的时候，会调用checkThread。
