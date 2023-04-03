# Glide框架详解

[链接]: https://blog.csdn.net/sinyu890807/category_9268670.html



#### Glide有几级缓存？

主要分成两大类共四级缓存，一个内存缓存，一个硬盘缓存。

* 活动缓存--内存存储：弱引用集合存储正在使用的图片，比如正在展示的图片，或者列表中不在当前页面展示的图片

* 内存缓存（LruCache）--内存缓存：存储最近被加载过的图片，存储在内存中，与活动缓存互斥，使用的是LRU机制

* 资源类（持久化缓存，DiskLruCache）--磁盘存储：存储被转换，解码后的图片，例如缩放过大小，圆角处理

* 数据类--磁盘存储：网络请求成功后原图存储到文件当中

  

**对于活动缓存和内存缓存互斥条件？**

对每张图片都有一个计数变量，每加载一次图片就会+1，每次release都会-1，如果值为0说明当前没有使用图片就会回收到第二层内存缓存中，如果大于0就会放入活动缓存中。

**为什么要设计两种内存缓存？**

例子：比如`Lru`缓存设置能缓存99张图片，当列表滑动到第100张图片时，那么就会回收掉已经加载的第一张图片，这样当滑动到第一张的时候需要重新去请求图片，这样显然没有充分利用资源。

所以增加了一个活动缓存来存储正在使用的图片，这样解决了二个问题：

* 活动缓内部数据结构为HashMap，比从内存缓存（内部数据结构LinkedHashMap），本地磁盘或重新加载获取图片都要更加高效
* 内存缓存内部实现机制是`LruCache`，当内存超出时会触发`TrimToSize()`去遍历链表并移除不需要使用缓存，链表遍历速度比较慢，比较消耗性能，增加了活动缓存分担`LruCache`压力，减少`TrimToSize`的概率，提升加载速度

* 虽然增加了活动缓存提高了内存占用，但活动缓存使用弱引用来存储图片，当系统内存不够时会释放内存，不会出现OOM。最差的情况也就是没有活动缓存，完全使用内存缓存和没有优化之前一样
  



#### Glide如何绑定生命周期？[链接](https://www.jianshu.com/p/b0b57cc2ede3)

* 通过获取`Activity`的`FragmentManager`，绑定一个空的`Fragment(RequestManagerFragment)`，用来同步`Activity`的生命周期
* `RequestManagerFragment`中初始化`Lifecycle`
  * `Lifecycle`是一个接口，实现类为`ActivityFragmentLifecycle`主要用于维护一个`LifecycleListener`集合，管理`LifecycleListener`，并用于生命周期的分发
* 初始化`RequestManagerFragment`的同时也会初始化`RequestManager`，并将`RequestManager`添加到`ActivityFragmentLifecycle`的集合中完成注册
  * `RequestManager`实现了`LifecyclerListener`接口，并在相应的生命周期中调用相关加载操作
  * 在生命周期`onStart`时继续加载，`onStop`时暂停加载，`onDestory`时停止加载任务和清除操作
* 当`Activity`触发生命周期时，会回调`RequestManagerFragment`的对应生命周期，透传至`Lifecycle`集合，遍历集合中所有已注册的`RequestManager`并调用相对应生命周期



**当页面有多Fragment嵌套如何实现生命周期的正确调用？**

![](..\images\Glide生命周期嵌套.jpg)

如图所示一个Activity采用`Viewpager + Fragment`的形式，而里面的`Fragment`又是一个`ViewPager + Fragment`的形式，这个时候，假设其中一个`RequestManagerFragment`生命周期方法走了，怎么知道哪些`RequestManagerFragment`绑定的`LifeCycle`应该得到调用呢？理想的情况是，应该让绑定该`RequestManagerFragment`的Fragment所有的子`Fragment`的`RequestManagerFragment`的生命周期得到调用

**Glide如何实现：**Glide会为`Activity`创建一个`RequestManagerFragment`做为`rootFragment`，并保存该`Activity`底下所有`Fragment`所创建的`RequestManagerFragment`，这些会保存在`RequestManagerTreeNode`类当中。

如图所示结构，当F1的生命周期被回调时，会遍历出当前`Activity`所有下`子Fragment`，并判断`子Fragment.getParentFragment`是否为`F1的Fragment`来找到对应的`F1`的子`Fragment`，并调用对应生命周期。



#### Glide如何实现Bitmap复用？ [链接](https://www.jianshu.com/p/7777948aef78)

**bitmap复用原理**

在 Android 3.0 上面引入了`BitmapFactory.Options.inBitmap`字段，如果设置了此选项，那么`Bitmap`在加载内容时尝试重复使用现有`bitmap`，不过`inBitmap `的使用方式不同版本下有一些不同，在 Android 4.4（API 级别 19）之前，系统仅支持大小相同的位图复用。在 Android 4.4 之后的版本，只要内存大小不小于需求的`Bitmap `都可以复用。



**Glide实现**

在 Glide 中，在每次解析一张图片为 Bitmap 的时候不管是内存缓存还是磁盘缓存，都会从其`BitmapPool `中查找一个可被复用的 Bitmap ，之后在将此块的内存缓存起来。具体实现类是`LruBitmapPool`，根据不同android版本`inBitmap`的实现不同使用策略模式用`AttributeStrategy `和`SizeConfigStrategy`两个类来实现`Bitmap`存储和获取。

* `AttributeStrategy `。通过 Bitmap 的 width（图片宽度）、height（图片高度） 和 config（图片颜色空间，比如 ARGB_8888 等） 三个参数作为 Bitmap 的唯一标识。当获取 Bitmap 的时候只有这三个条件完全匹配才行，支持4.4之前`Bitmap`复用。
* `SizeConfigStrategy`。使用 size（图片的像素总数） 和 config 作为唯一标识。当获取的时候会先找出 cofig 匹配的 Bitmap，然后保证该 Bitmap 的 size 大于我们期望的 size 并且小于期望 size 的 8 倍即可复用（可能是为了节省内存空间），支持4.4之后`Bitmap`复用。

**Bitmap复用好处：**复用图片会重复使用该图片的内存，并不能减少程序正在使用的内存大小，解决的是减少频繁申请内存带来的性能(抖动、碎片)问题。

详情原理
https://juejin.cn/post/6963675830639820814
https://toutiao.io/posts/ptdi4q/preview


**android中其他复用**

典型的比如 Handler 中的 Message. 当我们使用 Message 的 obtain 获取消息，实际上是从` Message Pool`中获取的，` Message Pool`内部结构是一个链表，当获取Message都是优先中`Message Pool`中查找可复用的内存，并不是直接new，这样实现原理和Glide的Bitmap复用其实是一样的。

