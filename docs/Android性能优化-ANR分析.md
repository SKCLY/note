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

* service timeout：前台服务在20s内未执行完成，后台服务10s
* broadcast timeout：前台广播在10s内未执行完成，后台60s
* contentprovider timeout：publish超过10s未执行完成
* inputdispatching timeout：输入事件分发超过5s，包括按键和触摸事件



**ANR触发原理**

service，broadcast，provider的原理都是一样的。

以service为例，service的onCreate方法调用之前会使用handler发送延时10s的消息，service的onCreate方法执行完会把这个延时消息移除掉。如果超过10s没有移除消息，延时消息就会正常执行，会收集信息生成trace文件，并弹窗提示。

input的超时检测机制稍微有点不同，是根据两次input事件的时候间隔来判断是否ANR。故一般input超时触发条件都是连续点击手机屏幕没有响应。



**ANR监控**

线下方法：

查看手机中`data/anr/traces.txt`文件，分析ANR原因。

线上方法：

[ANRWatchDog](https://github.com/SalomonBrys/ANR-WatchDog/)，主要实现原理如下：

1. 开启一个线程，死循环，循环中睡眠5s
2. 往UI线程post一个runnable，将_tick 赋值为0，将 _reported 赋值为false
3. 线程睡眠5s之后检查`_tick`和`_reported`字段是否被修改
4. 如果`_tick`和`_reported`没有被修改，说明给主线程post的Runnable一直没有被执行，也就说明主线程卡顿至少5s**（只能说至少，这里存在5s内的误差）**
5. 将线程堆栈信息输出

