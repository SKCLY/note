# Activity启动模式

手机的最近使用列表里存储的都是每个应用Task。每个Task都是对应应用Activity的回退栈，当应用完全退出最近任务列表不会清空。

* standard标准模式

​	    当应用打开第三方应用某个Activity时，这个Activity会直接加入当前应用的Task当中。（比如打开通讯录的新增联系人）

* singleTask 栈内复用模式

  当应用打开第三方应用某个Activity时，不会加入当前应用的Task，而会加入第三方应用自己的Task栈顶（没有打开就会创建一个新的Task），把整个Task放在当前应用Task上面。（比如新建邮件）

  如果调用的Activity存在，则不再创建直接复用，不会调用``onCreate()``方法，会调用``onNewIntent()``方法，并把Activity之上的所有Activity都清除。

* singleInstance 单例模式

  和SingleTask基本相同，区别是当前Task只有这一个Activity。

  应用场景：呼叫来电界面，调用系统联系人，锁屏页面。

* singleTop 栈顶复用模式

  把启动的``Activity``直接放入当前Task栈顶，会重用栈顶的`Activity`，重用时会调用`onNewIntent()`方法。

  应用场景：点击推送通知进入app
  
  

在Android是一个应用最多一个Task可以显示在最近任务列表中，如果两个Task的``taskAffinity``相同也只会在任务列表中展示一个。

在Android里每一个``Activity``都有一个``taskAffinity``，默认是取``Application``的``taskAffinity``，而Application默认是应用包名。

* 默认情况下打开`Activity`会直接进入当前Task
* 如果打开`Activity`配置了`SingleTask`，则会去对比`Activity`与当前Task的`taskAffinity`是否相同
  * 相同则直接入栈
  * 不相同会寻找与它`taskAffinity`相同的Task入栈，如果找不到会创建一个新的Task



实战通常的选择

* 默认(standard)和singleTop：多用于App内部
* singleInstance：多用于开放给外部App来共享使用
* singleTask：内部交互和外部交互都会用上



#### 当手机按下home键，发生了什么

