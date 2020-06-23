# Activity启动源码分析

Activity启动过程分2种，一种是应用程序根Activity启动也就是在XML布局中配置

``<action android:name="android.intent.action.MAIN"/> `` 属性的Activity。第二种程序中调用startActivity来跳转其他页面。

当我们点击某个应用的快捷图标的时候，就会通过 Launcher 请求 AMS 来启动该应用程序。

Launcher请求AMS的时序图如下：

![Launcher请求AMS的时序图](C:\DevelopmentTools\GitHub\note\images\Launcher请求AMS的时序图.png)













### Android进程模型与优先级

优化级从上往下逐级降低

``前端进程``  展示在屏幕上，并与用户交互的进程。

``可见进程``  展示在屏幕上非前端进程，如activity弹也一个对话框

``服务进程``  后台服务进程

``后台进程`` 已经被暂停的进程，例如执行onStop的activity，不会与其他进程进行交互

``空进程``  系统缓存机制，由zygote通过fork创建出来的进程被杀掉后留下空进程，创建其他线程时直接放入空进程中，提升创建效率

