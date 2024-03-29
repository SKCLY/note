# 性能优化-内存优化

#### 查看当前手机分配给一个APP内存大小

源代码中分配给一个APP初始内存是16M，但每个ROM都会有不同的配置信息

adb shell

getprop dalvik.vm.heapstartsize  app启动初始分配内存

getprop dalvik.vm.heapgrowthlimit  app最大内存限制

getprop  dalvik.vm.heapsize  开启largeHeap最大内存限制

#### Java对象生命周期



#### 如何判断一个对象是否可以回收？

引用计数法和可达性分析两种。

* 计算法

  通过计算一个对象被其他对象引用的次数来判断，如果没有其他对象与之关联，说明对象可以被回收。

  优点：实现简单，效率高

  缺点：无法解决循环引用的问题

* 可达性分析

  以`GC Roots`对象作为起点进行搜索，如果`GC Roots`和一个对象之间没有可达路径，则称对象是不可达的。
  
  `GC Roots`对象：
  
  * 静态变量引用的对象
  * 常量引用的对象
  * 线程栈引用对象
  * JNI引用对象
  
  

#### android垃圾回收机制

* 复制算法

  将可用的内存按容量分成大小相等两块，每次只使用其中一块。当一块用完了，将还存活的对象复制到另一块，并清除使用过的内存块。

  运行简单，效率高。但是可内存使用减少了一半。

  新生代对象回收非常快，需要复制的对象少，适合使用复制算法。

* 标记清除算法

  算法分为"标记"和“清除”两个阶段：首先标记出所有需要回收的对象，再进行统一回收被标记对象。

  标记清除之后会产生大量不连续的内存碎片，导致分配大对象时还是会无法找到足够连续内存，再次触发GC。

  回收的时候如果需要回收的对象越多，需要做的标记和清除的工作越多，所以标记清除算法适用于老年代。

* 标记整理算法

  标记出所有需要回收的对象，然后把存活的对象整理在一起，清理所有标记对象。和标记清除算法的区别就在于对存活对象的移动，可以减少内存碎片，但效率更加低下。

  故比较适合在老年代使用。



#### 内存三大问题

1. 内存抖动

   内存波动图呈锯齿状，GC卡顿。如自定义View在onDraw里创建对象
   
2. 内存泄露

   在当前应用周期内不再使用的对象被GC Roots引用，导致不能回收，使得实际可用内存变小

3. 内存溢出

   即OOM，OOM会导致程序异常。Android设备出厂后，java虚拟机对单个应用的最大内存分配就确定下来了，超出这个值就会OOM

#### Android内存泄漏常见问题及分析

1. 资源性对象未关闭

   对于资源对象不再使用时，应该立即调用它的close()函数，将其关闭并设置为null。例如Bitmap等资源未关闭会造成内存泄漏，应该在Activity销毁时及时关闭。

2. 注册对象未注销

   例如BroadcastReceiver，EventBus未注销造成内存泄漏，应该在Activity销毁时及时注销。

3. 类的静态变量持有大数据对象

   尽量避免使用静态变量存储数据，特别是大数据对象，建议使用数据库存储。

4. 单例造成的内存泄漏

   优先使用Application的Context，如需要使用Activity的Context，可以在传入Context时使用弱引用进行封装调用。

5. 非静态内部类的静态实例

   静态实例的生命周期和应用一样长，导致静态实例一直持有Activity引用，Activity内存资源不能正常回收。应该抽取出来封装成一个单例使用。

6. Handler临时性内存泄漏

   Message发出之后存储在MessageQueue中，在Message中存在target是Handler的一个引用，如果Handler是非静态内部类，Handler又会持有Activity的一个引用，故当Activity销毁时并不能正常GC，导致了内存泄漏。

   将handler变成静态内部类，但静态内部类是不能调用Activity内部非静态变量，故可以加上弱引用持有外部Activity来实现。

7. 容器中的对象没清理造成内存泄漏

   在退出程序之前，将集合里的东西clear并设置为null
   
8. 使用ListView时造成的内存泄漏

   在构造Adapter时使用缓存convertView

9. WebView

   使用单独进程，通过AIDL与主线程通信
   



#### Bitmap内存优化

##### Bitmap内存占用计算[链接](https://www.cnblogs.com/dasusu/p/9789389.html)

一般计算图片占用内存大小的公式为：**分辨率 * 像素点大小**

像素点大小的取决于图片的格式，android中图片格式如下

* ALPHA_8 -- 对应大小 1B
* RGB_565 -- 对应大小 2B
* ARGB_4444 -- 对应大小2B
* ARGB_8888 -- 对应大小4B，默认格式

故如果我们以一张分辨率为1080 * 452 的png格式的图片，图片本身大小为56kb

那么，这张图片的大小按照这个公式应该是：1080 * 452 * 4B = 1952640B ≈ 1.86MB

android项目中res目录下有不同的drawable目录，同样的图片放在不同目录下面又会对内存占用造成影响

因为系统在加载 res 目录下的资源图片时，会根据图片存放的不同目录做一次分辨率的转换，而转换的规则是：

新图的高度 = 原图高度 * (设备的 dpi / 目录对应的 dpi )

新图的宽度 = 原图宽度 * (设备的 dpi / 目录对应的 dpi )

目录名称与 dpi 的对应关系如下，drawable 没带后缀对应 160 dpi：

| 密度     | ldpi | mdpi(默认) | hdpi | xdpi | xxhdpi | xxxdpi |
| -------- | ---- | ---------- | ---- | ---- | ------ | ------ |
| 密度值   | 120  | 160        | 240  | 320  | 480    | 640    |
| 缩放系数 | 0.75 | 1          | 1.5  | 2    | 3      | 4      |

**图片位于res/drawable-hdpi**，设备dpi=240，设备1dp=1.5px

转换后的分辨率： 1080 * (240/240) * 452 * (240/240) = 1080 * 452

最终占用内存大小为：1080 * 452 * 4B = 1952640B（1.86MB）

**图片位于res/drawable-xhdpi**，设备dpi=240，设备1dp=1.5px

转换后的分辨率： 1080 * (240/320) * 452 * (240/320) = 810 * 339

最终占用内存大小为：810 * 339 * 4B = 1098360B（1.05MB）



需要按照正常图片尺寸放在不同的drawable文件夹里，如果需要尽量少的放文件夹，最好放在密度值大的文件夹里，这样不容易超成加载图片OOM。



##### Bitmap加载优化方式

1. 根据加载控件的大小调整读取图片的大小，不是全部把图片读取到内存中
2. 设备分级，需要根据不同的设备进行不同的处理，一般是对于低内存设备处理
   简单实现 `Application.isLowRamDeivce()`来判断如果是`lowRam`使用RGB_565，否则使用ARGB_8888
3. 使用图片缓存加载网络图片，减少网络请求
4. Android8之前Bitmap像素内存在`Java heap`中分配，android8之前手机内存比较小一般在1GB左右，故当`heap size`占用超过512M之后就比较容易`OOM`.anrdoid8之后Bitmap像素占用改为在`Native heap`中分配，而`Native heap`内存分配上限就很大，32 位应用的可用内存在3~4G，64位上更大，虚拟内存几乎很难耗尽，故android8之后手机使用Bitmap很难造成`OOM`。所以可以使用hook将android8之前Bitmap内存也放在`Native heap`层进行分配可大大减少`OOM`
   https://juejin.cn/post/7096059314233671694
   https://github.com/bytedance/android-inline-hook


#### Java中几种引用关系分别是什么，有什么区别？

* 强引用

  一般的Object obj = new Object()属性于强引用。只要强引用关联还在就不会被GC回收

* 软引用

  一些有用但并非必需数据，用软引用关联。系统在内存不够时会对软引用对象回收

* 弱引用

  一些有用程度比软引用更低数据，用弱引用关联。在系统发生GC时，不管内存够不够都会被回收

* 虚引用

  垃圾回收的时候收到一个通知，用来监控垃圾回收器是否正常工作，但是虚引用get返回的结果一直为null



#### LeakCanary检测工具

**原理**

[LeakCanary 内存泄漏原理完全解析](https://www.jianshu.com/p/59106802b62c)

![](..\images\leakcanary泄漏监听流程图.webp)

**如何完成Activity和Fragment的监听？**

构建一个`RefWatcher`对象用来监听回收，Activity通过`Application.registerActivityLifecycleCallbacks()`注册Activity的生命周期监听器，然后在`onActivityDestroyed`回调函数里监听Activity的回收状态。Fragment的`FragmentLifecycleCallbacks`是在`Activity.onActivityCreated()`回调里面进行注册的，在`onFragmentDestroyed`里面监听Framgment的回收状态。

监听对应的Activity和Fragment时都会创建对应一个key，因为同一个Activity和Framgent可以会创建多个实例，如果不用key进行区分的话，dump内存的时候会把有所实例都保存下来，会出来无法对应的情况。

**工作原理**

创建WeakReference对象引用Activity或Fragment，并指定一个ReferenceQueue的引用队列，如果Activity或Fragment被回收了，会将Acitvity或Fragment加入到ReferenceQueue队列中，故通过ReferenceQueue队列的`poll`方法遍历队列，可以取出所有弱引用对象。

**如何判断内存是否泄漏？**

leakcanary会单独开了一个线程`AndroidWatchExecutor `进行判断，主要判断逻辑如下

* 遍历弱引用队列取出对象的key，并将key从`retainedKeys`列表中移除。
* 判断当前对象的key是否包含在`retainedkeys`列表中，包含说明当前对象还没有被回收，不包含说明对象已经被回收。
* 如果当前对象没有进行回收，手动触发一次GC回收，再执行上两步的进行判断对象是否被回收，如果还没有回收，判断当前对象已经引发内存泄漏。
* 使用Debug类的dump内存中导出hprof文件构造一个HeadDump对象，建立导致内存泄漏的引用链。
