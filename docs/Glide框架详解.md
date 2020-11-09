# Glide框架详解

[链接]: https://blog.csdn.net/sinyu890807/category_9268670.html



#### Glide有几级缓存？

主要分成两个大的模块，一个内存缓存，一个硬盘缓存。

* 活动缓存（内存缓存）：弱引用集合存储正在使用的图片，比如在其他页面已经展示过的图片，
* 内存缓存（LruCache）：与活动缓存互斥，使用的是LRU机制
* 资源类型（持久化缓存，DiskLruCache）：存储被转换，解码后的图片，例如缩放过大小
* 数据来源（持久化缓存）：存储原图

对于活动缓存和内存缓存互斥条件？

对每张图片都有一个计数变量，每加载一次图片就会+1，每次release都会-1，如果值为0说明当前没有使用图片就会回收到LruCache中，如果大于0就会放入活动缓存中。

为什么要设计两种内存缓存？

1. 提高效率。LruCache实现是LinkedHashMap，活动缓存是用HashMap，读取效率高不少
2. 分担LruCache压力，减少TrimToSize的概率。使用弱引用当系统内存不够时会释放内存，不用担心OOM

#### Glide如何绑定生命周期？

Glide初始化的时候创建一个完全没有内容的fragment，在fragment里使用LifecycleCallback绑定生命周期，这样在Activity生命周期结束时就可以回收资源，从而不需要在Activity里面再去手动写回收资源代码。



