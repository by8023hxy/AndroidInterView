# 性能优化

### 1.内存优化

内存问题

- 内存泄漏
- 内存抖动：频繁创建临时对象
- Bitmap 大内存：规避位图超标
- 代码质量：intdef 代替枚举，使用 SparseArray 代替 HashMap

检测工具

- MAT(Memory Analysis Tools) ，可分析 Java 堆数据，可查看实例占用空间、引用关系等
- Android Studio 自带的 Profiler
- LeakCanary：通过弱引用和引用队列监控对象是否被回收，比如 Activity 销毁时开始监控此对象，检测到未被回收则主动 gc ，然后继续监控。

### 2.启动速度优化

##### 应用进程不存在的情况下，从点击桌面应用图标，到应用启动（冷启动），大概会经历以下流程：

1.Launcher startActivity

2.AMS startActivity

3.Zygote fork 进程

4.ActivityThread main()

  4.1. ActivityThread attach  

  4.2. handleBindApplication

  4.3 **attachBaseContext**

  4.4. installContentProviders

  4.5. **Application onCreate**

5.ActivityThread 进入loop循环

**6.Activity生命周期回调，onCreate、onStart、onResume...**

整个启动流程我们能干预的主要是 4.3、4.5 和6，应用启动优化主要从这三个地方入手。理想状况下，这三个地方如果不做任何耗时操作，那么应用启动速度就是最快的，但是现实很骨感，很多开源库接入第一步一般都是在Application onCreate方法初始化，有的甚至直接内置ContentProvider，直接在ContentProvider中初始化框架，不给你优化的机会。



#### 常见的启动优化方式大概有这些：

- 闪屏页优化
- <font color=#ff0000>MultipDex优化</font>
- 第三方库懒加载
- WebView优化
- 线程优化
- 系统调用优化



**MultiDex优化总结**

**方案1：直接在闪屏页开个子线程去执行MultiDex逻辑，MultiDex不影响冷启动速度，但是难维护。**

**方案2：今日头条的MultiDex优化方案：**

1. 在Application 的attachBaseContext 方法里，启动另一个进程的LoadDexActivity去异步执行MultiDex逻辑，显示Loading。
2. 然后主进程Application进入while循环，不断检测MultiDex操作是否完成
3. MultiDex执行完之后主进程Application继续走，ContentProvider初始化和Application onCreate方法，也就是执行主进程正常的逻辑。



### 3.绘制优化

- 减少布局层级及控件复杂度，避免过度绘制
- 使用 include、merge、viewstub
- 优化绘制过程，避免在 Draw 中频繁创建对象、做耗时操作

### 4.apk瘦身

1.资源方面：资源在线化、图片使用 webp 格式、tint 着色生成不同色调的切、使用 icon font

2.so 库：保留一个 cpu 架构的 so 文件

3.AS Inspect Code 清除无用代码和资源

4.代码混淆：使用 ProGuard 可以移除无用的类、字段、方法（压缩），移除无用字节码指令

5.不保留行号：使用 ProGuard 配置不保留行号

6.开启 shrinkResources：移除无用资源

7.资源混淆：使用 AndResGuard 缩短资源长度，对资源进行 7z 压缩等（直接对apk操作）

8.代码结构简化，比如用 intdef 代替 枚举(一个枚举有1~1.4kb大小)

9.使用 compileOnly 在只需编译时依赖的场景，不会打到 apk 里

10.使用 thinR 插件剔除 R 文件，将引用 R 字段的地方替换成对应常量

11.Android 7.0 使用 V2(apksigner) 代替 V1(jarsigner) 签名工具

12.动态加载 so 库(System.load加载绝对路径文件)、插件化技术、App Bundle

13.使用 facebook 的 redex



### 5.电量优化

#### 1、怎么做电量测试？

电量相关的测试相对来说难度较大，因为 App 在具体手机上的耗电量无法准确统计，每一个手机所使用的硬件不一样，那么它相应的功耗就不一样。而且这个功耗值我们只能在线下通过导出手机的 power_profile.xml 文件拿到。

由于我们无法获取准确的耗电量，所以我们只能增加多个维度来辅助判断 App 是否耗电。

最后，我们可以分场景各个突破。

关于电量测试，我们可以针对各个功能场景进行针对性的专项测试。操作一段时间后，我们可以在手机设置—电量消耗里面，利用其数据作为判断依据。这样虽然直观，但精确度不行。

介绍 Battery Historian：

- Google 推出的一款 Android 电量分析工具，它支持 Android 5.0 及以上系统的电量分析。
- 它获取到的各个耗电模块的耗电信息要相对精确、丰富地多。例如 GPS、WaleLock、蓝牙 等的工作时间以及耗电量。
- 此外，它不仅可以针对单个 App 进行选择，也可以比对不同的电量场景的信息，比如 优化前、优化后 的信息。
- Battery Historian 的缺点在于它只能在线下使用。因此除了使用其在线下测试之外，我们还需要在线上增加一些电量的辅助监控，统计例如：耗电组件的使用次数、调用堆栈以及访问时间。这些都是与用户相关的基础电量消耗数据，如果有用户反馈，我们就可以通过这些信息来判断用户是不是有耗电的操作。

#### 2、有哪些有效的电量优化手段？

因为我们不能在线上统计出 App 的电量消耗，因此需要在尽量保证 App 在正常使用下的耗电。对此我们采取了一系列的电量优化措施：

##### 1）、网络相关

- 网络请求的时机以及次数，将可以延迟的网络请求批量发送，减少网络被激活的时机与次数。
- 此外，我们可以对网络传输数据进行压缩，以降低传输的时间与流量。
- 最后，一定要禁止使用轮询的方式来做业务操作。

##### 2）、传感器相关

根据场景谨慎地选择传感器使用的模式，比如说在使用 GPS 的时候一般要避免使用高精度的模式，或者是尽量复用上一次的定位结果。

##### 3）、WakeLock

我们在实际项目中使用 WakeLock 有几个注意事项，第一，acquire、release 要成对地释放，第二，尽量使用 acquire 的超时方法来设置超时时间，避免因为异常情况从而导致 WakeLock 而无法释放的情况，第三，关于 WakeLock 的释放一定要写在 try-catch-finally 的 finally 当中，保证 WakeLock 在异常情况下的释放。

##### 4）、JobScheduler

JobScheduler 可以允许开发者在符合某些条件下创造执行在后台的任务，我们可以设置执行一些耗电操作的场景，比如说 处于 WIFI 状态下同时连接电源 的情况下。同时，要注意用户在离开界面后，要避免耗电的操作，比如说停止播放动画。通过这些操作，我们的 App 就不会比之前耗电了。

### 6.网络优化

尽量减少网络请求，能够合并的就尽量合并

避免 DNS 解析，根据域名查询可能会耗费上百毫秒的时间，也可能存在DNS劫持的风险。可以根据业务需求采用增加动态更新 IP 的方式，或者在 IP 方式访问失败时切换到域名访问方式。

大量数据的加载采用分页的方式

网络数据传输采用 GZIP 压缩

加入网络数据的缓存，避免频繁请求网络

上传图片时，在必要的时候压缩图片



## 7.内存泄漏场景及规避

**单例模式导致的内存泄漏。** 最常见的例子就是创建这个单例对象需要传入一个 Context，这时候传入了一个 Activity 类型的 Context，由于单例对象的静态属性，导致它的生命周期是从单例类加载到应用程序结束为止，所以即使已经 finish 掉了传入的 Activity，由于我们的单例对象依然持有 Activity 的引用，所以导致了内存泄漏。解决办法也很简单，不要使用 Activity 类型的 Context，使用 Application 类型的 Context 可以避免内存泄漏。

**静态变量导致的内存泄漏。** 静态变量是放在方法区中的，它的生命周期是从类加载到程序结束，可以看到静态变量生命周期是非常久的。最常见的因静态变量导致内存泄漏的例子是我们在 Activity 中创建了一个静态变量，而这个静态变量的创建需要传入 Activity 的引用 this。在这种情况下即使 Activity 调用了 finish 也会导致内存泄漏。原因就是因为这个静态变量的生命周期几乎和整个应用程序的生命周期一致，它一直持有 Activity 的引用，从而导致了内存泄漏。

**非静态内部类导致的内存泄漏。**非静态内部类导致内存泄漏的原因是非静态内部类持有外部类的引用，最常见的例子就是在 Activity 中使用 Handler 和 Thread 了。使用非静态内部类创建的 Handler 和 Thread 在执行延时操作的时候会一直持有当前Activity的引用，如果在执行延时操作的时候就结束 Activity，这样就会导致内存泄漏。解决办法有两种：第一种是使用静态内部类，在静态内部类中使用弱引用调用Activity。第二种方法是在 Activity 的 onDestroy 中调用 `handler.removeCallbacksAndMessages` 来取消延时事件。

使用资源未及时关闭导致的内存泄漏。常见的例子有：操作各种数据流未及时关闭，操作 Bitmap 未及时 recycle 等等。

使用第三方库未能及时解绑。有的三方库提供了注册和解绑的功能，最常见的就 EventBus 了，我们都知道使用 EventBus 要在 onCreate 中注册，在 onDestroy 中解绑。如果没有解绑的话，EventBus 其实是一个单例模式，他会一直持有 Activity 的引用，导致内存泄漏。同样常见的还有 RxJava，在使用 Timer 操作符做了一些延时操作后也要注意在 onDestroy 方法中调用 `disposable.dispose()`来取消操作。

属性动画导致的内存泄漏。常见的例子就是在属性动画执行的过程中退出了 Activity，这时 View 对象依然持有 Activity 的引用从而导致了内存泄漏。解决办法就是在 onDestroy 中调用动画的 cancel 方法取消属性动画。

WebView 导致的内存泄漏。WebView 比较特殊，即使是调用了它的 destroy 方法，依然会导致内存泄漏。其实避免WebView导致内存泄漏的最好方法就是让WebView所在的Activity处于另一个进程中，当这个 Activity 结束时杀死当前 WebView 所处的进程即可，我记得阿里钉钉的 WebView 就是另外开启的一个进程，应该也是采用这种方法避免内存泄漏。



#### 扩大内存

为什么要扩大我们的内存呢？有时候我们实际开发中不可避免的要使用很多第三方商业的 SDK，这些 SDK 其实有好有坏，大厂的 SDK 可能内存泄漏会少一些，但一些小厂的 SDK 质量也就不太靠谱一些。那应对这种我们无法改变的情况，最好的办法就是扩大内存。

扩大内存通常有两种方法：一个是在清单文件中的 Application  下添加`largeHeap="true"`这个属性，另一个就是同一个应用开启多个进程来扩大一个应用的总内存空间。第二种方法其实就很常见了，比方说我使用过个推的 S DK，个推的 Service 其实就是处在另外一个单独的进程中。

Android 中的内存优化总的来说就是开源和节流，开源就是扩大内存，节流就是避免内存泄漏。



## 8.图片加载如何避免 OOM

我们知道内存中的 Bitmap 大小的计算公式是：长所占像素 * 宽所占像素 * 每个像素所占内存。想避免 OOM 有两种方法：等比例缩小长宽、减少每个像素所占的内存。

- 等比缩小长宽。我们知道 Bitmap 的创建是通过 BitmapFactory 的工厂方法，`decodeFile()、decodeStream()、decodeByteArray()、decodeResource()`。这些方法中都有一个 Options 类型的参数，这个 Options 是 BitmapFactory 的内部类，存储着 BItmap 的一些信息。Options 中有一个属性：inSampleSize。我们通过修改 inSampleSize 可以缩小图片的长宽，从而减少 BItma p 所占内存。需要注意的是这个 inSampleSize 大小需要是 2 的幂次方，如果小于 1，代码会强制让inSampleSize为1。
- 减少像素所占内存。Options 中有一个属性 inPreferredConfig，默认是 `ARGB_8888`，代表每个像素所占尺寸。我们可以通过将之修改为 `RGB_565` 或者 `ARGB_4444` 来减少一半内存。

### 大图加载

加载高清大图，比如清明上河图，首先屏幕是显示不下的，而且考虑到内存情况，也不可能一次性全部加载到内存。这时候就需要局部加载了，Android中有一个负责局部加载的类：BitmapRegionDecoder。使用方法很简单，通过BitmapRegionDecoder.newInstance()创建对象，之后调用decodeRegion(Rect rect, BitmapFactory.Options options)即可。第一个参数rect是要显示的区域，第二个参数是BitmapFactory中的内部类Options。







