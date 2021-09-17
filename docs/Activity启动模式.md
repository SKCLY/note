# Activity启动模式

手机的最近任务列表里存储的都是每个应用Task。每个Task都是对应应用Activity的回退栈，当应用完全退出最近任务列表不会清空，会保留一个残影，主要是方便用户进入应用，此时点击应用会重新启动应用。

* `standard`标准模式

​	    在不同应用中打开其他应用的同一个Activity，Activity会被创建多个实例，分别放进每个Task当中。

​		例子：A应用打开通讯录的新增加联系人页面，此时点击返回按钮返回到A应用。此时打开通讯录页面启动的通讯录列表页面，而不是新增加联系人页面。

* `singleTask` 栈内复用模式

  当Activity被其他应用启动时不会进入启动它应用的Task里，而是会在属于它自己的Task里面创建，并放置在Task的栈顶（如果在Task中已经有Activity，则直接复用并把该Activity上面的所有Activity都清除，保证自己在栈顶并调用`onNewIntent()`），并把整个Task压到启动它应用Task上面。

  例子：A应用打开邮箱APP的新建邮件页面，会在邮箱APP的Task栈中找是否已经创建了新建邮件页面，如果有把该页面上面所有Activity清除，放在栈顶；如果没有创建一个新的页面放在栈顶面，然后把整个Task叠加到A应用的Task上面。

  * 为什么这么说呢，如果这个时候按返回键会回退到邮箱APP的之前页面中，直到邮箱应用页面全部被退出，才会返回到A应用中。
  * 但是这种Task相互叠加的只有前台Task才有，一旦用户按Home键或查看最近任务列表键，叠加的Task就会进行拆分，这时再点击创建邮件页面再连接返回直到邮箱应用完全退出，就会直接返回到桌面，而不是A应用。

* `singleInstance` 单例模式

  行为逻辑基本上和SingleTask是相同的，区别是`singleInstance`只允许当前Task只有这一个Activity。

  例子：A应用打开邮箱APP的新建邮件页面(现在当前页面为`singleInstance`模式)，此时会新建一个Task里面只有新建邮件一个页面并压到A应用的Task上面，此时按返回键会直接回到A应用。如果新建邮件页面再打开新的Activity，并不会在当前Task中添加新的Activity，而是会打开邮箱APP的Task创建新的Activity并压到当前Task上面。

  这个时候如果按查看最近任务列表键，会发现后面任务列表中并没有新建邮件Task，而是只有邮箱APP的Task。因为这两个Task的`taskAffinity`相同了，所以在最近任务列表中只会展示一个。

  应用场景：呼叫来电界面，锁屏页面。

* `singleTop` 栈顶复用模式

  行为逻辑和`standard`比较像，会把启动的``Activity``直接放入当前Task栈顶，如果此是栈顶的Activity正好是需要启动的Activity，那就会复用这个Activity，并调用`onNewIntent()`。

  应用场景：点击推送通知进入app的页面
  



**taskAffinity**

在Android里每一个``Activity``都有一个``taskAffinity``，默认是取``Application``的``taskAffinity``，而Application的`taskAffinity`默认是应用包名。

* 默认情况下打开`Activity`会直接进入当前Task
* 如果打开`Activity`配置了`SingleTask`，则会去对比`Activity`与当前Task的`taskAffinity`是否相同
  * 相同则直接入栈
  * 不相同会寻找与它`taskAffinity`相同的Task入栈，如果找不到会创建一个新的Task

所以按照上面所说，如果你仔细观察会发现当你有APP中启动一个配置了`singleTask`的Activity，这个Activity如果来自别的APP切换是应用间动画效果，如果是自己应用中的Activity，就是普通页面跳转效果。那是因为启动其他应用的Activity时`taskAffinity`是不同的，那么Activity创建之后会放在别的Task栈顶，再切换就会发生Task的切换。

在Android里一个应用最多一个Task可以显示在最近任务列表中，如果两个Task的``taskAffinity``相同也只会在任务列表中展示一个。



**实战通常的选择**

* 默认(standard)和singleTop：多用于App内部
* singleInstance：多用于开放给外部App来共享使用
* singleTask：内部交互和外部交互都会用上
