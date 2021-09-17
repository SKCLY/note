# 崩溃分析

[卡顿、死锁、ANR原理，线上监控方案分析](https://www.jianshu.com/p/df257ff1bfec)



1. 第三方对接  bugly，友盟等
2. 本地日志系统
   1. 实现 Thread.UncaughtExceptionHandler 接口，在uncaughtException可以捕获异常，打印并记录下来
   2. 对接阿里云日志服务上传日志信息，方便查找异常。信息内容如下
      * 发生时间
      * 手机型号
      * Android系统版本
      * 应用版本号
      * 异常信息栈
      * 用户行为（记录接口信息，点击页面）
      * onTrimMemory()方法内去释放部分内存
   3. 最好是后台服务日志对比查看
3. 查看trace.txt文件



#### ANR原理

**哪些场景会造成ANR**

* service timeout：前台服务在10s内未执行完成，后台服务20s
* broadcast timeout：前台广播在10s内未执行完成，后台60s
* contentprovider timeout：publish超过10s未执行完成
* inputdispatching timeout：输入事件分发超过5s，包括按键和触摸事件



**ANR触发原理**

service，broadcast，provider的原理都是一样的。

以service为例，service的onCreate方法调用之前会使用handler发送延时10s的消息，service的onCreate方法执行完会把这个延时消息移除掉。如果超过10s没有移除消息，延时消息就会正常执行，会收集信息生成trace文件，并弹窗提示。

input的超时检测机制稍微有点不同，是根据两次input事件的时候间隔来判断是否ANR。故一般input超时触发条件都是连续点击手机屏幕没有响应。

代码分析：

应用启动时在`ActivityThread`里面调用`looper.loop`开启循环，在主线程执行任务的流程：通过主线程Handler post一个任务到消息队列里去，在loop循环中通过`MessageQueue.next()`从队列中拿到`Message`，再调任`handler.dispatchMessage`处理消息。

这里可以看到可能出现卡顿有两个地方：

* `messageQueue.next()`获取消息

从代码中可以看到`next`方法是不断从MessageQueue里取出消息，有消息就处理，没有消息就调用`nativePollOnce`阻塞，并且阻塞线程会进入休眠并释放CPU时间片(通过native层实现)，故不会引起ANR。

* `handler.dispatchMessage`处理消息

处理就是重写`handler.handleMessage`方法，在这里交给用户处理，故很容易因为处理消息太耗时导致ANR。



**ANR监控**

线下方法：

查看手机中`data/anr/traces.txt`文件，分析ANR原因。

线上方法：

* 使用FileObserver去监听data/anr/这个目录，如果当前目录有创建文件，说明有新的anr产生，可以上传anr文件用于分析anr。

[ANRWatchDog](https://github.com/SalomonBrys/ANR-WatchDog/)

参照上面框架，设计一个anr监控工具原理如下：

1. 定义两个全局变量 _tick 和 _reported 
2. 自定义一个线程，开启死循环，将_tick进行赋值
3. 创建UI线程的handler，往UI线程post一个`runnable`，`runnable`内主要执行就是两句：将_tick 赋值为0，将 _reported 赋值为false
4. 循环中睡眠一定时间，默认设置为5s
5. 线程睡眠之后检查`_tick`和`_reported`字段是否被修改
6. 如果`_tick`和`_reported`没有被修改，说明给主线程post的Runnable一直没有被执行，也就说明主线程卡顿至少5s**（只能说至少，这里存在5s内的误差）**
7. 通过`Looper.getMainLooper().getThread().getStackTrace()`来获取当前的堆栈信息
8. 通过一个`ANRListener`接口返回错误信息，可以给开发者定义，默认实现直接抛出异常
