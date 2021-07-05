# Activity启动源码分析

Activity启动过程分2种，一种是应用程序根Activity启动也就是在XML布局中配置

``<action android:name="android.intent.action.MAIN"/> `` 属性的Activity。第二种程序中调用startActivity来跳转其他页面。

当我们点击某个应用的快捷图标的时候，就会通过 Launcher 请求 AMS 来启动该应用程序。

Launcher请求AMS的时序图如下：

![Launcher请求AMS的时序图](..\images\Launcher请求AMS的时序图.png)



### Android进程模型与优先级

优化级从上往下逐级降低

``前端进程``  展示在屏幕上，并与用户交互的进程。

``可见进程``  展示在屏幕上非前端进程，如activity弹也一个对话框

``服务进程``  后台服务进程

``后台进程`` 已经被暂停的进程，例如执行onStop的activity，不会与其他进程进行交互

``空进程``  系统缓存机制，由zygote通过fork创建出来的进程被杀掉后留下空进程，创建其他线程时直接放入空进程中，提升创建效率



### ActivityManagerService

[完整链接](https://juejin.im/post/5dc4339c5188254e7a15585c)

Activity任务栈模型

![image](..\images\Activity任务栈模型.png)

1.ActivityRecord作用就是记录Activity所有信息，包括 AMS 的引用、AndroidManifes 节点信息、Activity 状态、Activity 资源信息和 Activity 进程相关信息等，需要注意的是其中包含ActivityRecord所在的TaskRecord。

2.TaskRecord用来描述一个Activity任务栈，包括任务栈的唯一标识符、任务栈的倾向性、任务栈中的 Activity 记录和 AMS 的引用等，需要注意的是其中含有 ActivityStack。

3.ActivityStack是一个管理类，用来管理系统所有Activity，其内部维护了Activity所有状态，特殊状态的Activity和Activity相关的列表等数据。