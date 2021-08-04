### Flutter面试



### android项目引入Flutter开发

在 Android Studio 打开现有的 Android 项目并点击菜单按钮 File > New > New Module… ，这样就可以创建出一个可以集成的新 Flutter 模块，或者选择导入已有的 Flutter 模块。

![](..\images\引入flutter项目1.png)

如果你想创建一个新的 Flutter 模块，则可以直接在向导窗口中填写模块名称、路径等信息。

![](..\images\引入flutter项目2.png)



### Flutter路由配置

Flutter路由表配置是在主入口main.dart里面的MaterialApp里面进行，主要有两个入口
1.  routes表路径跳转到相应的页面。
2.  如果在routes项里面没有找到对应路由，会在`onGenerateRoute`方法会有一个setttings参数，里面存储有路径和参数。

`routes`一个键值对表，里面key对应的路由名称，value返回的对应页面，这样需要跳转哪个页面只需要传对应的key值就可以跳转了。但是如果当页面比较多，大家都修改main.dart文件容易出错，故一般都会单独一个路由表类进行路由配置，这样就要用到`onGenerateRoute`方法。

`onGenerateRoute`方法有一个setting参数里面存储有跳转页面路径和参数，当`routes`里面没有找到对应路由就会调用`onGenerateRoute`方法，这里可以调用统一路由配置工具类。

```dart
new MaterialApp(
      routes: {
       '/home':(BuildContext context) => HomePage(),
       '/home/one':(BuildContext context) => OnePage(),
      },
      onGenerateRoute: (setting) {
          Uri uri = Uri.parse(settings.name);
          // 获取跳转页面route
          String route = uri.path;
          //获取跳转页面带参数
          Map<String, String> params = uri.queryParameters;
      }
);
```

### Flutter与原生交互

在Android当中创建一个Activity继承FlutterActivity或者FlutterFragmentActivity，在跳转到Flutter页面时会先打开这个Activity然后把Flutter页面加载到这个Activity里面，有点类似于WebView的工作原理。
flutter和native之间的数据交互是双向，其中作为通讯桥梁的是MethodChannel，这个类在初始化的时候需要注册一个渠道值，这个值必需是唯一的且native层和flutter层要一样。

**Flutter跳转android**

Flutter端实现：

````dart
class FlutterToNative {
  //通过KEY_CHANNEL注册一个MethodCall
  static const String KEY_CHANNEL = "flutterMethod";
  var _methodChannel = MethodChannel(KEY_CHANNEL);
  // 传递给native一个method和参数
  navNativePage(String key) {
    return _invokePlatformMethod("testNative", params: {'text': text});
  }
   //传递给native一个method获取数据
  getNativeInfo() {
    return _invokePlatformMethod("userInfo");
  }

  Future<dynamic> _invokePlatformMethod(String methodName, {dynamic params}) async {
    var result;
    try {
      result = await _methodChannel.invokeMethod(methodName, params);
    } on PlatformException catch (e) {
      print('平台方法返回错误 : ${e}');
    }
    debugPrint("返回结果: " + result.toString());
    return result;
  }
}
````

Android端实现：

````java
public class FlutterPageActivity extends FlutterFragmentActivity {
    public static final String CHANNEL_NATIVE = "flutterMethod";
    private MethodChannel methodChannel;

    @Override
    public void configureFlutterEngine(@NonNull FlutterEngine flutterEngine) {
        super.configureFlutterEngine(flutterEngine);
        //通过CHANNEL注册MethodChannel
        methodChannel = new MethodChannel(flutterEngine.getDartExecutor(), CHANNEL_NATIVE);
        //注册回调函数
        methodChannel.setMethodCallHandler(new MethodChannel.MethodCallHandler() {
            @Override
            public void onMethodCall(@NonNull MethodCall call, @NonNull MethodChannel.Result result) {
            	//flutter端的传递的method会在这里遍历，可以进行页面跳转和数据查询
                switch (call.method) {
                    case "testNative":
                        Intent intent = new Intent(FlutterPageActivity.this, ToNativeActivity.class);
                        final String text = call.argument("text");
                        intent.putExtra("text", text);
                        startActivity(intent);
                        break;

                    case "userInfo":
                        HashMap<String, String> arguments = new HashMap<String, String>();
                        arguments.put("name", "sksk");
                        arguments.put("age", "123");
                        arguments.put("amount", "12.33");
         				//可以进行查询数据通过reuslt返回
                        result.success(arguments);
                        break;
                }
            }
        });
    }
}
````



**android跳转Flutter**

Android端跳转实现：

````java
Intent intent = new Intent(MainActivity.this, FlutterPageActivity.class);
intent.putExtra("route", "/flutterPage?name=heizi&key=123");
startActivity(intent);
````

Flutter端实现：

```dart
new MeterialApp(
    onGenerateRoute: (setttings) {
          Uri uri = Uri.parse(settings.name);
          //uri为路由的值，这里为/flutterPage
          String route = uri.path;
          //路由后面带的参数，这里为{name: sk, key: 123}
          Map<String, String> params = uri.queryParameters;
    }
)
```



### Flutter网络请求配置

网络请求是基于[dio](https://github.com/flutterchina/dio/blob/master/README-ZH.md)三方库进行封装开发的。dio是一个强大的Dart Http请求库，支持Restful API、FormData、拦截器、请求取消、Cookie管理、文件上传/下载、超时、自定义适配器等。

### Flutter常用三方控件

* [dio](https://github.com/flutterchina/dio/blob/master/README-ZH.md)网络加载框架
* EasyLoading 加载样式
* [pull_to_refresh](https://github.com/peng8350/flutter_pulltorefresh/blob/master/README_CN.md)下拉刷新框架
* [fsuper](https://github.com/Fliggy-Mobile/fsuper/blob/master/README_CN.md)富文本
* [flutter_picker](https://github.com/yangyxd/flutter_picker)时间选择框