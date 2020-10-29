# 性能优化-内存优化

#### 查看当前手机分配给一个APP内存大小

源代码中分配给一个APP初始内存是16M，但每个ROM都会有不同的配置信息

adb shell

getprop dalvik.vm.heapstartsize  app启动初始分配内存

getprop dalvik.vm.heapgrowthlimit  app最大内存限制

getprop  dalvik.vm.heapsize  开启largeHeap最大内存限制

#### Java对象生命周期



#### android垃圾回收机制

* 标记清除算法

  老年代使用

* 复制算法

  新生代使用

* 标记整理算法

  青年代使用

#### android 低内存杀进程机制



#### 内存三大问题

1. 内存抖动

   内存波动图呈锯齿状，GC卡顿。如自定义View在onDraw里创建对象
   
2. 内存泄露

   在当前应用周期内不再使用的对象被GC Roots引用，导致不能回收，使得实际可用内存变小

3. 内存溢出

   即OOM，OOM会导致程序异常。Android设备出厂后，java虚拟机对单个应用的最大内存分配就确定下来了，超出这个值就会OOM

#### Android内存泄漏常见问题及分析

1. Bitmap占用内存优化
   1. 设备分级，需要根据不同的设备进行不同的处理，一般是对于低内存的设备处理
   2. 统一图片加载工具，根据图片大小和显示图片控件调整
