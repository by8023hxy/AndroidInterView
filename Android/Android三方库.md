# 1.OkHttp

##### 这个库是做什么用的？

网络底层库，它是基于http协议封装的一套请求客户端，虽然它也可以开线程，但根本上它更偏向真正的请求，跟HttpClient, HttpUrlConnection的职责是一样的。其中封装了网络请求get、post等底层操作的实现。

##### 为什么要在项目中使用这个库？

- OkHttp 提供了对最新的 HTTP 协议版本 HTTP/2 和 SPDY 的支持，这使得对同一个主机发出的所有请求都可以共享相同的套接字连接。
- 如果 HTTP/2 和 SPDY 不可用，OkHttp 会使用连接池来复用连接以提高效率。
- OkHttp 提供了对 GZIP 的默认支持来降低传输内容的大小。
- OkHttp 也提供了对 HTTP 响应的缓存机制，可以避免不必要的网络请求。
- 当网络出现问题时，OkHttp 会自动重试一个主机的多个 IP 地址。

##### 这个库都有哪些用法？对应什么样的使用场景？

get、post请求、上传文件、上传表单等等。

##### 这个库的优缺点是什么，跟同类型库的比较？

- 优点：在上面
- 缺点：使用的时候仍然需要自己再做一层封装。

##### 这个库的核心实现原理是什么？如果让你实现这个库的某些核心功能，你会考虑怎么去实现？

OkHttp内部的请求流程：使用OkHttp会在请求的时候初始化一个Call的实例，然后执行它的execute()方法或enqueue()方法，内部最后都会执行到getResponseWithInterceptorChain()方法，这个方法里面通过拦截器组成的责任链，依次经过用户自定义普通拦截器、重试拦截器、桥接拦截器、缓存拦截器、连接拦截器和用户自定义网络拦截器以及访问服务器拦截器等拦截处理过程，来获取到一个响应并交给用户。其中，除了OKHttp的内部请求流程这点之外，缓存和连接这两部分内容也是两个很重要的点，掌握了这3点就说明你理解了OkHttp。

##### 各个拦截器的作用：

- interceptors：用户自定义拦截器
- retryAndFollowUpInterceptor：负责失败重试以及重定向
- BridgeInterceptor：请求时，对必要的Header进行一些添加，接收响应时，移除必要的Header
- CacheInterceptor：负责读取缓存直接返回（根据请求的信息和缓存的响应的信息来判断是否存在缓存可用）、更新缓存
- ConnectInterceptor：负责和服务器建立连接

ConnectionPool：

1、判断连接是否可用，不可用则从ConnectionPool获取连接，ConnectionPool无连接，创建新连接，握手，放入ConnectionPool。

2、它是一个Deque，add添加Connection，使用线程池负责定时清理缓存。

3、使用连接复用省去了进行 TCP 和 TLS 握手的一个过程。

- networkInterceptors：用户定义网络拦截器。
- CallServerInterceptor：负责向服务器发送请求数据、从服务器读取响应数据。

# 2.Retrofit

##### 这个库是做什么用的？

Retrofit 是一个 RESTful 的 HTTP 网络请求框架的封装。Retrofit 2.0 开始内置 OkHttp，前者专注于接口的封装，后者专注于网络请求的高效。



##### 为什么要在项目中使用这个库？

1、功能强大：

- 支持同步、异步
- 支持多种数据的解析 & 序列化格式
- 支持RxJava

2、简洁易用：

- 通过注解配置网络请求参数
- 采用大量设计模式简化使用

3、可扩展性好：

- 功能模块高度封装
- 解耦彻底，如自定义Converters

##### 这个库都有哪些用法？对应什么样的使用场景？

任何网络场景都应该优先选择，特别是后台API遵循Restful API设计风格 & 项目中使用到RxJava。

##### 这个库的优缺点是什么，跟同类型库的比较？

- 优点：在上面
- 缺点：扩展性差，高度封装所带来的必然后果，如果服务器不能给出统一的API形式，会很难处理。

##### 这个库的核心实现原理是什么？如果让你实现这个库的某些核心功能，你会考虑怎么去实现？

Retrofit主要是在create方法中采用动态代理模式（通过访问代理对象的方式来间接访问目标对象）实现接口方法，这个过程构建了一个ServiceMethod对象，根据方法注解获取请求方式，参数类型和参数注解拼接请求的链接，当一切都准备好之后会把数据添加到Retrofit的RequestBuilder中。然后当我们主动发起网络请求的时候会调用okhttp发起网络请求，okhttp的配置包括请求方式，URL等在Retrofit的RequestBuilder的build()方法中实现，并发起真正的网络请求。

##### 你从这个库中学到什么有价值的或者说可借鉴的设计思想？

内部使用了优秀的架构设计和大量的设计模式，在我分析过Retrofit最新版的源码和大量优秀的Retrofit源码分析文章后，我发现，要想真正理解Retrofit内部的核心源码流程和设计思想，首先，需要对它使用到的九大设计模式有一定的了解，下面我简单说一说：

1、创建Retrofit实例：

- 使用建造者模式通过内部Builder类建立了一个Retroift实例。
- 网络请求工厂使用了工厂方法模式。

2、创建网络请求接口的实例：

- 首先，使用外观模式统一调用创建网络请求接口实例和网络请求参数配置的方法。
- 然后，使用动态代理动态地去创建网络请求接口实例。
- 接着，使用了建造者模式 & 单例模式创建了serviceMethod对象。
- 再者，使用了策略模式对serviceMethod对象进行网络请求参数配置，即通过解析网络请求接口方法的参数、返回值和注解类型，从Retrofit对象中获取对应的网络的url地址、网络请求执行器、网络请求适配器和数据转换器。
- 最后，使用了装饰者模式ExecuteCallBack为serviceMethod对象加入线程切换的操作，便于接受数据后通过Handler从子线程切换到主线程从而对返回数据结果进行处理。

3、发送网络请求：

- 在异步请求时，通过静态delegate代理对网络请求接口的方法中的每个参数使用对应的ParameterHanlder进行解析。

4、解析数据

5、切换线程：

- 使用了适配器模式通过检测不同的Platform使用不同的回调执行器，然后使用回调执行器切换线程，这里同样是使用了装饰模式。

6、处理结果

# 3.Glide

Glide是一个Android的图片加载和缓存库，它主要专注于大量图片的流畅加载。是google所推荐的图片加载库，作者是bumptech。这个库被广泛运用在google的开源项目中，包括2014年的google I/O大会上发布的官方App。



**Glide特点**

1、多样化媒体加载

Glide 不仅是一个图片缓存，它支持 Gif、WebP等格式

2、生命周期集成

我们可以更加高效的使用Glide提供的方式进行绑定，这样可以更好的让加载图片的请求的生命周期动态管理起来

3、高效的缓存策略

- 支持Memory和Disk图片缓存
- 根据 ImageView 的大小来缓存相应大小的图片尺寸
- 内存开销小，默认的 Bitmap 格式是 RGB_565 格式（3.X版本），4.7.1版本默认格式为（PREFER_ARGB_8888_DISALLOW_HARDWARE）
- 使用BitmapPool进行Bitmap的复用

4、 提供丰富的图片转换Api，支持圆形裁剪、平滑显示等特性



**Fresco：**

- 最大的优势在于5.0以下(最低2.3)的bitmap加载。在5.0以下系统，Fresco将图片放到一个特别的内存区域(Ashmem区)
- 大大减少OOM（在更底层的Native层对OOM进行处理，图片将不再占用App的内存）
- 适用于需要高性能加载大量图片的场景

对于一般App来说，Glide完全够用，而对于图片需求比较大的App，为了防止加载大量图片导致OOM，Fresco 会更合适一些。并不是说用Glide会导致OOM，Glide默认用的内存缓存是LruCache，内存不会一直往上涨。

#### **总体设计**

1、构建Request，实现类为SingleRequest，用于发起一个加载的请求

2、通过EngineJob和DecodeJob负责任务创建，发起，回调，资源的管理

3、根据请求的资源类型，最后匹配对应的DateFetcher进行Data数据的获取

4、获取数据进行相应的缓存配置

5、根据原始数据Data进行解码及转换，生成最终需要显示的Resource

6、通过回调Target对应的方法，最后进行图片的显示

**Glide是如何与Activity及Fragment等的生命周期绑定？**

Glide在执行with的阶段，会根据context的类型，将Glide的Request请求与context类型进行绑定。Application类型为整个应用的生命周期。Fragment及Activity类型，通过巧妙的设计一个RequestManagerFragment,加入到Activity或Fragment当中，从而实现生命周期的监听。

**Glide的缓存实现原理是怎样的？**

Glide的缓存使用内存缓存及硬盘缓存进行处理。

| 缓存            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| ActiveResources | ActiveResources是一个以弱引用资源为value。用于缓存正在使用的资源 |
| MemoryCache     | MemoryCache是使用LruResourceCache实现,用于缓存非正在使用的资源 |
| DiskCache       | 进行资源磁盘缓存                                             |
| Http            | 通过网络地址，从服务端加载资源文件                           |

**假如用户配置了使用内存缓存及磁盘缓存，则主要的加载实现流程如下:**

1、当发起Request时，首先会从ActiveResources中进行缓存查找。如果命中则返回显示，如果不命中，则从MemoryCache中获取。当资源从ActiveResources中移除后，加入到MemoryCache中

2、当在MemoryCache中命中时，则会将资源加入到ActiveResources中，并在该Cache中移除，如果不命中则会尝试从磁盘缓存中进行加载

3、根据用于配置的策略，如果在磁盘缓存中命中，则会返回，并将资源缓存到ActiveResources当中，如果不命中，则会将进行网络的请求

4、根据ModelLoader的配置实现，从网络中加载资源，并根据配置，缓存到磁盘及内存缓存中

**内存缓存**

**Glide主要的内存缓存策略采用了2级缓存，为ActiveResources和MemoryCache。**



#### **硬盘缓存**

**缓存策略**

**Glide缓存的资源分为两种（1，原图（SOURCE）原始图片 2，处理图（RESULT）经过压缩和变形等转化的图片）**

硬盘缓存分为五种，具体看一面。可以通过调用diskCacheStrategy()方法并传入五种不同的参数

1，DiskCacheStrategy.NONE// 表示不缓存任何内容

2，DiskCacheStrategy.DATA// 表示只缓存原始图片

3，DiskCacheStrategy.RESOURCE// 表示只缓存转换过后的图片

4，DiskCacheStrategy.ALL // 表示既缓存原始图片，也缓存转换过后的图片

5，DiskCacheStrategy.AUTOMATIC//表示让Glide根据图片资源智能地选择使用哪一种缓存策略（默认选项）



**Glide的底层网络实现是什么？**

Glide默认的网络加载使用的是urlConnection。当然我们也可以通过自定义ModelLoader，使用okhttp、volley等的网络框架进行加载

**Glide中代码运用了那些设计模式，有什么巧妙的设计？**

1、建造者模式

Glide对象的创建使用Build模式，将复杂对象的创建和表示分离，调用者不需要知道复杂的创建过程，使用Build的相关方法进行配置创建对象。

2、外观模式

Glide对外提供了统一的调度，屏蔽了内部的实现，使得使用该网络库简单便捷。

3、策略模式

关于DecodeJob中的DataFetcherGenerator资源获取，采用了策略模式，将数据加载的不同算法进行封装。

4、工厂模式

ModelLoader的创建使用了ModelLoaderFactory、Engine中的EngineJobFactory、DiskLruCacheFactory等



### **假如让你自己写个图片加载框架，你会考虑哪些问题？**

异步加载：最少两个线程池

切换到主线程：Handler

缓存：LruCache、DiskLruCache，涉及到LinkHashMap原理

防止OOM：软引用、LruCache、图片压缩没展开讲、Bitmap像素存储位置源码分析、Fresco部分源码分析

内存泄露：注意ImageView的正确引用，生命周期管理

列表滑动加载的问题：加载错乱用tag、队满任务存在则不添加。

# 4.Rxjava

**rxjava是ReactiveX在java平台的一个实现。是一个编程模型，以观察者模式提供链式的接口调用，动态控制线程的切换，使得可以简便的处理异步数据流。**

**Rxjava特点**

- 链式调用，使用简单
- 简化逻辑
- 灵活的线程调度
- 提供完善的数据操作符，功能强大

从RxJava在github上面的官方介绍来说，RxJava是一个在Java虚拟机上的响应式扩展，通过使用可观察的序列将异步和基于事件的程序组合起来的一个库。

它扩展了观察者模式来支持数据/事件序列，并且添加了操作符，这些操作符允许你声明性地组合序列，同时抽象出要关注的问题：比如低级线程、同步、线程安全和并发数据结构等。

简单点来说， RxJava就是一个使用了观察者模式，能够异步的库。

举个例子，以微信公众号为例，一个微信公众号会不断产生新的内容，如果我们读者对这个微信公众号的内容感兴趣，就会订阅这个公众号，当公众号有新内容时，就会推送给我们。我们收到新内容时，如果是我们感兴趣的，就会点进去看下;如果是广告的话，就可能直接忽略掉。这就是我们生活中遇到的典型的观察者模式。

在上面的例子中，微信公众号就是一个被观察者(Observable)，不断的产生内容（事件），而我们读者就是一个观察者(Observer) ，通过订阅（subscribe）就能够接受到微信公众号（被观察者）推送的内容（事件），根据不同的内容（事件）做出不同的操作。

**2.1Rxjava角色说明**

RxJava的扩展观察者模式中就是存在这么4种角色：

| 角色                   | 角色功能                   |
| ---------------------- | -------------------------- |
| 被观察者（Observable） | 产生事件                   |
| 观察者（Observer）     | 响应事件并做出处理         |
| 事件（Event）          | 被观察者和观察者的消息载体 |
| 订阅（Subscribe）      | 连接被观察者和观察者       |

**2.2****RxJava事件类型**

RxJava中的事件分为三种类型：Next事件、Complete事件和Error事件。具体如下：

| 事件类型 | 含义     | 说明                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| Next     | 常规事件 | 被观察者可以发送无数个Next事件，观察者也可以接受无数个Next事件 |
| Complete | 结束事件 | 被观察者发送Complete事件后可以继续发送事件，观察者收到Complete事件后将不会接受其他任何事件 |
| Error    | 异常事件 | 被观察者发送Error事件后，其他事件将被终止发送，观察者收到Error事件后将不会接受其他任何事件 |

**3.****RxJava的消息订阅**

1.创建被观察者(Observable),定义要发送的事件。

2.创建观察者(Observer)，接受事件并做出响应操作。

3.观察者通过订阅（subscribe）被观察者把它们连接到一起。

**4.****创建被观察者过程**

![image-20211205105058501](/Users/baiyu/Library/Application Support/typora-user-images/image-20211205105058501.png)

Observable(被观察者)和Observer(观察者)建立连接(订阅)之后，会创建出一个发射器CreateEmitter，发射器会把被观察者中产生的事件发送到观察者中去，观察者对发射器中发出的事件做出响应处理。可以看到，是订阅之后，Observable(被观察者)才会开始发送事件。

![image-20211205105124218](/Users/baiyu/Library/Application Support/typora-user-images/image-20211205105124218.png)

**5.Rxjava线程切换原理**

 RxAndroid 就是通过 Handler 来拿到主线程的，用handler来发送Message去实现的，感兴趣的可以看下。 既然ObserveOnObserver实现了Runnable接口，那么就是其run()方法会在主线程中被调用。AndroidSchedulers.mainThread()是通过消息将run方法的实现交由主线程Looper进行处理，达到将观察者的数据处理在主线程中执行的效果

**Scheduler**

我们在调用subscribeOn与observeOn时，都会传入Scheduler对象，首先我们先看一下Scheduler的种类及其功能

| Scheduler种类                     | 说明                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| Schedulers.io( )                  | 用于IO密集型的操作，例如读写SD卡文件，查询数据库，访问网络等，具有线程缓存机制，在此调度器接收到任务后，先检查线程缓存池中，是否有空闲的线程，如果有，则复用，如果没有则创建新的线程，并加入到线程池中，如果每次都没有空闲线程使用，可以无上限的创建新线程 |
| Schedulers.newThread( )           | 在每执行一个任务时创建一个新的线程，不具有线程缓存机制，因为创建一个新的线程比复用一个线程更耗时耗力，虽然使用Schedulers.io( )的地方，都可以使用Schedulers.newThread( )，但是，Schedulers.newThread( )的效率没有Schedulers.io( )高 |
| Schedulers.computation()          | 用于CPU 密集型计算任务，即不会被 I/O 等操作限制性能的耗时操作，例如xml,json文件的解析，Bitmap图片的压缩取样等，具有固定的线程池，大小为CPU的核数。不可以用于I/O操作，因为I/O操作的等待时间会浪费CPU |
| Schedulers.trampoline()           | 在当前线程立即执行任务，如果当前线程有任务在执行，则会将其暂停，等插入进来的任务执行完之后，再将未完成的任务接着执行 |
| Schedulers.single()               | 拥有一个线程单例，所有的任务都在这一个线程中执行，当此线程中有任务执行时，其他任务将会按照先进先出的顺序依次执行 |
| Scheduler.from(Executor executor) | 指定一个线程调度器，由此调度器来控制任务的执行策略           |
| AndroidSchedulers.mainThread()    | 在Android UI线程中执行任务，为Android开发定制                |

这里会通过维护一个Worker的线程池来达到IoScheduler线程复用的效果

**6.rxjava背压策略实现原理是怎样的？**

当上下游在不同的线程中，通过Observable发射，处理，响应数据流时，如果上游发射数据的速度快于下游接收处理数据的速度，这样对于那些没来及处理的数据就会造成积压，这些数据既不会丢失，也不会被垃圾回收机制回收，而是存放在一个异步缓存池中，如果缓存池中的数据一直得不到处理，越积越多，最后就会造成内存溢出，这便是响应式编程中的背压（backpressure）问题。