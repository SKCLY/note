# 页面优化

#### APP启动耗时统计

应用启动状态：

* 冷启动。是指应用从头开始启动：系统进程在冷启动后才创建应用进程。发生冷启动的情况包括应用设备启动后或系统终止应用后首次启动。
* 热启动。指应用所有Activity都驻留内存中，系统的所有工作只是将Activity带到前台。一般home按键快速切换应用
* 温启动。包含了在冷启动期间发生的操作，同时开销要比热启动高。
  * 用户点击返回键退出应用。进程可能还存在，只是应用需要重新创建页面。
  * APP由于内存不足被回收。可以使用已保存的实例savedInstanceState加快一部分启动



启动时间统计指令： adb shell am start -S -W  [packageName]/.[activityName]

命令行启动应用后会出现三个时间

ThisTIme：表示一连串启动Activity的最后一个Activity启动的耗时

TotalTime：表示新应用启动的耗时，包括新进程的启动和Activity的启动，但不包括前一个应用的pause的耗时

WaitTIme：包括前一个应用Activity pause的时间和新应用启动的时间

一般用TotalTime来查看应用冷启动耗时，一般比较理想的是1s左右。



启动优化的路径： application整个启动流程，MainActivity的onCreate()和onWindowFouceChange()两个方法。



#### CPU Profile工具使用

需要在android studio里面项目的run configurations里面的profiling标签里面进行配置选择Trace Java Method，不过这个只支持android 8.0以上的版本。

可以使用安卓自带Debug工具类

```java
在Application中添加如下代码
public class AppApplication extends BaseApplication {

    @Override
    public void onCreate() {
        super.onCreate();
        // 后面的字符串就是trace文件的存储目录
        Debug.startMethodTracing("/sdcard/sk3");
    }
}

在启动Activity中添加如下代码
public class MainActivity extends AppCompatActivity {
    @Override
    public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        Debug.stopMethodTracing();
    }
}
```

这样APP启动以后就会在对应目录生成trace文件，将trace文件导入Profiler工具就可以看到APP启动详细时间。

Profiler里面有三种图表可以用来分析APP启动时间

Flame Chart

提供一个倒置的调用图表，用来汇总APP的调用堆栈，调用顺序是从下往上。横轴显示的是百分比数值，可以展示每次调用消耗时间占整个启动时长的百分比，也就是说长度越长启动时间越长，有可能是需要优化的地方。

Top Down Tree

显示一个调用列表，这个列表中展开方法节点会显示它调用的方法节点，并且每个节点都有精确的时间信息，可以精确量化优化的效果。

有三个时间信息

* Self Time：运行自己代码消耗的时间
* Children Time：调用其他方法的时间
* Total Time：前面两者时间之和



应用中页面跳转时启动页面所需要时间，一般在logcat中会有输出，不同机型输出日志不同

* 小米，三星，Vivo：ActivityManager: Displayed，一般格式为`ActivityManager: Displayed [packageName]/.[activityName]: +562ms`
* 华为：ScreenCommon



#### StrictMode严苛模式

StrictMode是一个开发人员工具，它可以检测出我们可能无意中做的事情，并将它们提请我们注意，以便我们能够 修复它们。

````java
public class MyApplication extends Application {
    @Override
    public void onCreate() {
        if (BuildConfig.DEBUG) {
            //线程检测策略
            StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                    .detectDiskReads()   //读、写操作
                    .detectDiskWrites()
                    .detectNetwork()   // or .detectAll() for all detectable problems
                    .penaltyLog()
                    .penaltyDeath()
                    .build());
            StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                    .detectLeakedSqlLiteObjects()   //Sqlite对象泄露
                    .detectLeakedClosableObjects()  //未关闭的Closable对象泄露
                    .penaltyLog()  //违规打印日志
                    .penaltyDeath() //违规崩溃
                    .build());
        }

        super.onCreate();
    }
}
````



#### 启动速度优化建议


