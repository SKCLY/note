# Glide框架详解

[链接]: https://blog.csdn.net/sinyu890807/category_9268670.html



#### Glide有几级缓存？

主要分成两个大的模块，一个内存缓存，一个硬盘缓存。

* 活动缓存：弱引用集合存储正在使用的图片，存储在内存中，比如正在展示的图片，或者列表中不在当前页面展示的图片。

* 内存缓存（LruCache）：存储最近被加载过的图片，存储在内存中，与活动缓存互斥，使用的是LRU机制

* 磁盘缓存-资源类型（持久化缓存，DiskLruCache）：存储被转换，解码后的图片，例如缩放过大小，圆角处理

* 磁盘缓存-数据来源（持久化缓存）：网络请求成功后原图存储到文件当中

  

**对于活动缓存和内存缓存互斥条件？**

对每张图片都有一个计数变量，每加载一次图片就会+1，每次release都会-1，如果值为0说明当前没有使用图片就会回收到第二层内存缓存中，如果大于0就会放入活动缓存中。

**为什么要设计两种内存缓存？**

**`LruCache`内部实现机制**：使用的是`LruCache`，内部是用一个`Set`来缓存对象的，每次内存超出时触发`trimToSize`操作，会对`Set`进行遍历并移除缓存，这有可能把正在使用的缓存给删除了。

例子：比如`Lru`缓存设置能缓存99张图片，当列表滑动到第100张图片时，那么就会回收掉已经加载的第一张图片，这样当滑动到第一张的时候需要重新去请求图片，这样显然没有充分利用资源。

所以增加了一个活动缓存来存储正在使用的图片，这样解决了二个问题：

* 加载正在使用中的图片效率问题。
  * 原来需要重新加载或者从本地磁盘中读取，现在直接从活动缓存中读取，效率更加高
  * `LruCache`是用LinkedHashMap存储数据，活动缓存优化为HashMap，提升读取效率
  * `TrimToSize()`需要遍历链表，速度比较慢。增加了活动缓存分担`LruCache`压力，减少`TrimToSize`的概率，提升加载速度

* 本质是用空间换时间的策略。 活动缓存使用弱引用来存储图片，当系统内存不够时会释放内存，不用担心占用太多的内存导致OOM



#### Glide如何绑定生命周期？[链接](https://www.jianshu.com/p/b0b57cc2ede3)

`Glide.with(this)`绑定了Activity的生命周期，在Activity内创建了一个无UI的Fragment(`RequestManagerFragment`)，这个Fragment持有一个`ActivityFragmentLifecycle`，在Fragment关键生命周期通知`RequestManager`进行相关从操作。在生命周期`onStart`时继续加载，`onStop`时暂停加载，`onDestory`时停止加载任务和清除操作。



#### Glide的BitmapPool如何实现复用？ [链接](https://www.jianshu.com/p/7777948aef78)

**bitmap复用原理**

在 Android 3.0 上面引入了`BitmapFactory.Options.inBitmap`字段，如果设置了此选项，那么`Bitmap`在加载内容时尝试重复使用现有位图。这样可以复用现有的`Bitmap `，减少对象创建，从而减少发生 GC 的概率。不过，`inBitmap `的使用方式存在一些不同，在 Android 4.4（API 级别 19）之前，系统仅支持大小相同的位图进行复用。在 Android 4.4 之后的版本，只要内存大小不小于需求的`Bitmap `都可以复用。



**Glide实现**

在 Glide 中，在每次解析一张图片为 Bitmap 的时候不管是内存缓存还是磁盘缓存，都会从其`BitmapPool `中查找一个可被复用的 Bitmap ，之后在将此块的内存缓存起来。内部实现`Bitmap`复用的类是`LruBitmapPool`，根据不同android版本`inBitmap`的不同实现使用策略模式实现了`AttributeStrategy `和`SizeConfigStrategy`两个类来实现`Bitmap`存储和获取。

* `AttributeStrategy `。通过 Bitmap 的 width（图片宽度）、height（图片高度） 和 config（图片颜色空间，比如 ARGB_8888 等） 三个参数作为 Bitmap 的唯一标识。当获取 Bitmap 的时候只有这三个条件完全匹配才行，因为4.4之前仅支持大小相同的Bitmap复用。
* `SizeConfigStrategy`。使用 size（图片的像素总数） 和 config 作为唯一标识。当获取的时候会先找出 cofig 匹配的 Bitmap，然后保证该 Bitmap 的 size 大于我们期望的 size 并且小于期望 size 的 8 倍即可复用（可能是为了节省内存空间）。

**好处：**在使用复用池的时候，如果存在能被复用的图片会重复使用该图片的内存。 所以复用并不能减少程序正在使用的内存大小，解决的是减少频繁申请内存带来的性能(抖动、碎片)问题。



**android中其他复用**

典型的比如 Handler 中的 Message. 当我们使用 Message 的 obtain 获取消息，实际上是从 Message 池中获取的。Handler 中的 Message 是通过链表维护的数据结构，以此来构成一个 “Message 池”。这个池的最大数量由 `MAX_POOL_SIZE` 这个参数指定，即为 50。

