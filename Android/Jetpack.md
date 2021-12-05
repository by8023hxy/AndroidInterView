# 1.简述一下Jetpack相关的组件：

Lifecycle ：能够帮我们轻松的应对 Activity/Fragment 的生命周期问题，能够让我们以一种更加解耦的方式处理生命周期的变化问题，以及轻松避免内存泄露。

LiveData ：基于观察者模式、并且感知生命周期的数据持有类，能够帮助我们更好地解耦与处理数据。

ViewModel + Data Binding ：为我们在 Android 平台上实现 MVVM 架构提供了非常有效而强大的支持。

Room ：提供了一种更加友好高效的数据库持久化的功能。

WorkManager ：为我们执行后台任务提供了一站式解决方案。

Navigation ：能够帮助我们更加方便地构建单 Activity 应用。

Paging ：能够帮助我们应对加载大数据问题。

DataStore:是一种改进的新数据存储解决方案，允许使用**协议缓冲区**存储键值对或类型化对象。`DataStore` **以异步、一致的事务方式存储数据，克服了 SharedPreferences（以下统称为SP）的一些缺点**。

Splashscreen:高效打造自由、丰富的应用启动效果



# 2.Jetpack LiveData是什么？它的出现解决了什么问题?

LiveData 是一个能够感知生命周期、可观察的数据持有类 ，它被设计成 ViewModel 的一个成员变量；可以以一个 更解耦 的方式来共享数据。

LiveData 的实现基于**观察者**模式；

LiveData 跟 LifecycleOwner 绑定，能**感知生命周期变化**，并且只会在 LifecycleOwner 处于 Active 状态（STARTED/RESUMED）下通知数据改变；

LiveData 会自动在 DESTROYED 的状态下移除 Observer ，取消订阅，所以**不用担心内存泄露**；

LiveData 的存在，主要是为了帮助 新手老手 都能不假思索地遵循 通过唯一可信源分发状态 的标准化开发理念，从而使在快速开发过程中 难以追溯、难以排查、不可预期 的问题所发生的概率降低到最小。

# 3.**Jetpack Lifecycle** 是什么？它为什么被称为AAC的基石？

### **Lifecycle 是一个专门用来处理生命周期的库，它能够帮助我们将 Acitivity、Framgent 的生命周期处理与业务逻辑处理进行完全解耦，让我们能够更加专注于业务；通过解耦让 Activity、Fragment 的代码更加可读可维护。**

#### 可以这么说 **Lifecycle 的出现彻底解决了 Android 开发遇到的生命周期处理难题**，并且还给开发者带来了新的架构姿势，让我们可以设计出更加合理的架构。

##### 1.Lifecycle 库通过在 SupportActivity 的 onCreate 中注入 ReportFragment 来感知发生命周期；

##### 2.Lifecycle 抽象类，是 Lifecycle 库的核心类之一，它是对生命周期的抽象，定义了生命周期事件以及状态，通过它我们可以获取当前的生命周期状态，同时它也奠定了观察者模式的基调；（我是党员你看出来了吗:-D）

##### 3.LifecycleOwner ，描述了一个拥有生命周期的组件，可以自己定义，不过通常我们不需要，直接使用 AppCompatActivity 等即可；

##### 4.LifecycleRegistry 是 Lifecycle 的实现类，它负责接管生命周期事件，同时也负责 Observer 的注册以及通知；

##### 5.ObserverWithState ，是 Observer 的一个封装类，是它最终 通过 ReflectiveGenericLifecycleObserve 调用了我们用注解修饰的方法；

##### 6.LifecycleObserver ，Lifecycle 的观察者，利用它我们可以享受 Lifecycle 带来的能力；

##### 7.ReflectiveGenericLifecycleObserver，它存储了我们在 Observer 里注解的方法，并在生命周期发生改变的时候最终通过反射的方式调用对应的方法。



# 4**.Jetpack ViewModel**

<font color=#ff0000>ViewModel 的存在，主要是为了解决 状态管理 和 页面通信 的问题</font>

ViewModel 的本职工作是 **状态托管** 和 **状态管理的分治**，也即当视图控制器重建时，

对于轻量的状态，可以通过视图控制器基类的 saveInstanceState 机制，以序列化的方式完成存储和恢复。

对于重量级的状态，例如通过网络请求得到的 List，可以通过生命周期长于视图控制器的 ViewModel 持有，从而得以直接从 ViewModel 恢复，而不是以效率较低的序列化方式。

在 Jetpack ViewModel 面市之前，MVP 的 Presenter 和 MVVM - Clean 的 ViewModel 都不具备状态管理分治的能力。

Presenter 和 Clean ViewModel 的生命周期都与视图控制器同生共死，因而它们顶多是为 DataBinding 提供状态的托管，而无法实现状态的分治。

到了 Jetpack 这一版，ViewModel 以精妙的设计，达成了状态管理，以及可共享的作用域。

#### **ViewModel 为什么能做到这几点？**

其实这版主要是基于 **工厂模式**，使得 ViewModel **被 LifecycleOwner 所持有、通过 ViewModelProvider 来引用**，

所以 **它既类似于单例：** —— 当被作为 LifecycleOwner 的 Activity 持有时，能够脱离 Activity 旗下 Fragment 的生命周期，从而实现作用域共享，

**实际上又不是单例：** —— 生命周期跟随 作为 LifecycleOwner 的视图控制器，当 Owner（Activity 或 Fragment）被销毁时，它也被 clear。

**ViewModel 的目标只有三个：**

1.让状态管理独立于视图控制器，从而做到重建状态的分治、状态在多页面的共享，以及跨页面通信。

2.为状态设置作用域，使状态的共享做到作用域可控。

3.实现单向依赖，避免内存泄漏。



# 5.**Jetpack DataBinding**



DataBinding 的存在，主要是为了解决 视图调用 的一致性问题。

**DataBinding 只负责绑定数据、负责作为 UI 逻辑末端的状态的改变**（也即它是一个不可再分的原子操作，本来就不需要调试），原本在视图控制器中 UI 逻辑怎么写，现在还是怎么写，只不过不再需要 textView.setText(xxx)，而是直接 xxx.set()。

所以在 DataBinding 的帮助下，好处总共有多少个呢？

1.规避了视图状态的 一致性问题 —— 无需手工判空。

2.规避了视图状态的 一致性问题，乃至无需视图调用，从而完全不用编写 findViewById。

3.就算要调用视图，也不用 findViewById，而是直接通过 binding 来引用。

4.先前的 UI 逻辑基本不用改动，改的只是作为末端的状态改变的方式。





#### 总结:

**Lifecycle 的存在，主要是为了解决 生命周期管理 的一致性问题。**

**LiveData 的存在，主要是为了帮助 新手老手 都能不假思索地 遵循 通过唯一可信源分发状态 的标准化开发理念，从而在快速开发过程中 规避一系列 难以追溯、难以排查、不可预期 的问题。**

**ViewModel 的存在，主要是为了解决 状态管理 和 页面通信 的问题。**

**DataBinding 的存在，主要是为了解决 视图调用 的一致性问题。**

**它们的存在 大都是为了 在软件工程的背景下 解决一致性的问题、将容易出错的操作在后台封装好，方便使用者快速、稳定、不产生预期外错误地编码。**

