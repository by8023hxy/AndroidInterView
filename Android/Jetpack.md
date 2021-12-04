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