# Context完全解析

Context继承结构图

![image](..\images\Context的继承结构.png)

1. 从图中可以看出Activity和Service以及Application的Context是不一样的,Activity继承自ContextThemeWraper.其他的继承自ContextWrapper。

2. 由于Context的具体实现由ContextImpl实现，因此大部分场景下Activity，Service和Application三种类型的Context可以通用。不过在几种特殊场景下需要注意，比如启动Activity，还有弹出Dialog。出于安全原因Android规定Activity和Dialog的创建必须建立在一个Activity的基础上，故只能使用Activity的Context，否则报错。

3. Context的数量。Context有Application，Service，Activity三种，数量公式为`Activity数量+Serviceo数量+Application(1个)`

   #### getApplication()和getApplicationContext()区别？

   两种方法返回的内容都是Application的实例，主要区别在作用域。 getApplication()只有在Activity和Service中才能调用到，在其他场景无法使用。getApplicationContext()作用域更大，除了上述的范围，在BroadcastReceiver中也能使用。

   #### getBaseContext()方法有什么作用？ 

   getBaseContext()得到是一个ContextImpl对象，Context具体功能都是由ContextImpl实现，getResources()、getPackageName()、getSystemService()等常用方法都是调用ContextImpl实现的。如何实现：ContextWrapper里面有一个attachBaseContext()方法会传入ContextImpl实例，从而将实现委托给ContextImpl，故调用Context方法只有attachBaseContext()执行之后才能执行，否则会报错，因为这个时候ContextWrapper还没有持有ContextImpl的实例。

#### 如何正确使用Application？

google官方建议，一般不需要自定义Application，如果需要也只做一些全局初始化的工作，如果需要全局工具类可以使用单例模式。

#### ApplicationContext和ActivityContext的区别？

首先这是两种不同的context，ApplicationContext继承自ContextWrapper，ActivityContext继承自ContextThemeWrapper。

ApplicationContext的生命同期与Application相关，对于一般应用来说applicationContext创建和销毁相当于应用的创建和关闭。ActivityContext的生命周期与Activity相关，activity可以有多次创建，多次销毁。故使用context需要注意如下事项

* 不要让生命周期长的对象引用ActivityContext，保证Activivty销毁时ActivityContext也要销毁，需要会引起OOM
* 对于长生命周期的对象，可以使用ApplicationContext替代