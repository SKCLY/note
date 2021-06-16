# Android历代版本

### Android 11 [链接](https://juejin.cn/post/6948211914455384072)



#### 1.Scoped Storage（分区存储）

不过需要注意的是，应用`targetSdkVersion >= 30`，强制执行分区存储机制。之前在`AndroidManifest.xml`中添加 `android:requestLegacyExternalStorage="true"`的适配方式已不起作用，相当于Android11需要强制使用scoped storage。

**MANAGE_EXTERNAL_STORAGE权限**

如果你的应用是手机管家、文件管理器这类需要访问大量文件的app，可以申请`MANAGE_EXTERNAL_STORAGE`权限，将用户引导至系统设置页面开启。

代码如下：

````java
<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" tools:ignore="ScopedStorage" />
    
public static void checkStorageManagerPermission(Context context) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R && !Environment.isExternalStorageManager()) {
        Intent intent = new Intent(Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(intent);
    }
}
````

一般应用使用MediaStore和ASF就能满足所有场景，该权限是给一些特殊应用使用的，类似于文件浏览器或文件管理类应用。

我们可以使用`ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION`这个action来跳转到指定的授权页面，可以通过`Environment.isExternalStorageManager()`这个函数来判断用户是否已授权。



#### 2.权限变化

**单次权限授权**

从 Android 11 开始，每当应用请求与位置信息、麦克风或摄像头相关的权限时，面向用户的权限对话框会包含仅限这一次选项。如果用户在对话框中选择此选项，系统会向应用授予临时的单次授权。

单次权限授权的应用可以在一段时间内访问相关数据，具体时间取决于应用的行为和用户的操作：

* 当应用的 Activity 可见时，应用可以访问相关数据
* 如果用户将应用转为后台运行，应用可以在短时间内继续访问相关数据
* 如果在 Activity 可见时启动了一项前台服务，并且用户随后将应用转到后台，那么应用可以继续访问相关数据，直到该前台服务停止
* 如果用户撤消单次授权（例如在系统设置中撤消），无论是否启动了前台服务，应用都无法访问相关数据。与任何权限一样，如果用户撤消了应用的单次授权，应用进程就会终止

当用户下次打开应用并且应用中的某项功能请求访问位置信息、麦克风或摄像头时，系统会再次提示用户授予权限。



**请求位置权限**

请求`ACCESS_FINE_LOCATION`或 `ACCESS_COARSE_LOCATION`权限表示在前台时拥有访问设备位置信息的权限。在请求弹框中，选择“始终允许”表示前后台都可以获取位置信息，选择“仅在应用使用过程中允许”只表示拥有前台的权限。

建议：

* 先请求前台位置信息访问权限，再请求后台位置信息访问权限
* 单独请求后台位置信息访问权限，不要与其他权限一同请求



**软件包的可见性**

软件包可见性是Android 11上提升系统隐私安全性的一个新特性。它的作用是限制app随意获取其他app的信息和安装状态。避免病毒软件、间谍软件利用，引发网络钓鱼、用户安装信息泄露等安全事件。

主要影响是在`targetSdkVersion >= 30`中，第三方应用默认都是不可见的。类似方法`getInstalledPackages`、`getPackageInfo`返回数据都不包含第三方应用，影响比较多的是分享支付一类需要与其他应用交互的功能。

解决方法很简单，在`AndroidManifest.xml` 中添加`queries`元素，里面添加需要可见的应用包名。

````java
<manifest package="com.example.app">
    <queries>
		<!-- 微博 -->
        <package android:name="com.sina.weibo" />
        <- 指定微信包名 ->
        <package android:name="com.tencent.mm" />
        <!-- QQ -->
        <package android:name="com.tencent.mobileqq" />
        <!-- 支付宝 -->
        <package android:name="com.eg.android.AlipayGphone" /> 
        <!-- AlipayHK -->
        <package android:name="hk.alipay.wallet" />
    </queries> 
</manifest>
````

有一点需要说明一下，我们日常使用的`startActivity` 方法不受系统软件包可见性行为的影响。如果我们跳转是直接写死对应包名是不受影响，只有在做跳转前进行类似`hasActivity`的判断，才会受影响。



**前台服务类型**

Android 10中，在前台服务访问位置信息，需要在对应的`service`中添加 `location` 服务类型。

同样的，Android 11中，在前台服务访问摄像头或麦克风，需要在对应的`service`中添加`camera`或`microphone` 服务类型。

````java
<manifest>
   <service 
       android:name="MyService"
       android:foregroundServiceType="microphone|camera" />
</manifest>
````

这一限制的变更，使得程序无法在后台启动服务访问摄像头和麦克风。如需使用，只能是前台开启前台服务。除非有如下情况：

- 服务由系统组件启动。
- 服务是通过应用小部件启动。
- 服务是通过与通知交互启动的。
- 服务是`PendingIntent`启动的，它是从另一个可见的应用程序发送过来的。
- 服务由一个应用程序启动，该应用是一个[DPC](https://developer.android.google.cn/work/dpc/build-dpc)，且在设备所有者模式下运行。
- 服务由一个提供`VoiceInteractionService`的应用启动。
- 服务由一个具有`START_ACTIVITIES_FROM_BACKGROUND`权限的应用启动。



**权限自动重置**

如果应用以 Android 11 或更高版本为目标平台并且数月未使用，系统会通过自动重置用户已授予应用的运行时敏感权限来保护用户数据。

可能通过引导用户关闭自动重置功能，代码如下：

````java
public void checkAutoRevokePermission(Context context) {
	// 判断是否开启
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R && !context.getPackageManager().isAutoRevokeWhitelisted()) {
        // 跳转设置页
        Intent intent = new Intent(Intent.ACTION_AUTO_REVOKE_PERMISSIONS);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setData(Uri.fromParts("package", context.getPackageName(), null));
        context.startActivity(intent);
    }
}
````



**读取手机号码**

如果你是通过`TelecomManager`的`getLine1Number`方法，或`TelephonyManager`的`getMsisdn`方法获取电话号码。那么在Android 11中需要增加`READ_PHONE_NUMBERS`权限。使用其他方法不受限。

````java
<manifest>
    <!-- 如果应用仅在 Android 10及更低版本中使用该权限，可以添加 maxSdkVersion="29" -->
    <uses-permission android:name="android.permission.READ_PHONE_STATE"
                     android:maxSdkVersion="29" />
    <uses-permission android:name="android.permission.READ_PHONE_NUMBERS" />
</manifest>
````



**状态栏高度**

发现系统为Android 11的手机上`targetSdkVersion` 是30时获取状态栏高度为0，低于30获取值正常。。。因此需要使用`WindowMetrics `适配一下：

````java
public static int getStatusBarHeight(Context context) {
 	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
        WindowManager wm = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
        WindowMetrics windowMetrics = wm.getCurrentWindowMetrics();
        WindowInsets windowInsets = windowMetrics.getWindowInsets();
        Insets insets = windowInsets.getInsetsIgnoringVisibility(WindowInsets.Type.navigationBars() | WindowInsets.Type.displayCutout());
        return insets.top;
    }
}
//WindowMetrics是Android 11新增的类，用于获取窗口边界，同样可以用来获取导航栏高度。
````



**兼容性调试工具**

以往我们做适配的时候，需要先将我们项目中的 `targetSdkVersion` 修改为对应版本。这就导致你适配过程中有可能受到其他变更的影响，而这个新增的兼容性调试工具可以让你在不升级`targetSdkVersion`的情况下，针对每项变更逐个开启适配。

使用方法：

- 开发者选项中找到应用兼容性变更选项。
- 点击进入找到你需要调试的应用
- 在变更列表中，找到想要开启或关闭的变更，然后点击相应的开关。

![](C:\DevelopProject\learn\note\images\Android11兼容工具.jpg)





### Android 10

#### 1. Scoped Storage（分区存储）

在Android 10之前的版本上，我们在做文件的操作时都会申请存储空间的读写权限。但是这些权限完全被滥用，造成的问题就是手机的存储空间中充斥着大量不明作用的文件，并且应用卸载后它也没有删除掉。为了解决这个问题，Android 10 中引入了Scoped Storage 的概念，通过添加外部存储访问限制来实现更好的文件管理。

首先明确一个概念，外部储存和内部储存

* 内部储存：`/data`目录。一般我们使用`getFilesDir()`或`getCacheDir()`方法获取本地应用的内部储存路径，读写该路径下的文件不需要申请储存空间读写权限，且卸载时会自动删除。
* 外部储存：`/storage`或`/mnt`目录。一般使用`getExternalStorageDirectory()`方法获取储存路径。



![](..\images\Android10 scoped storage.png)

如上图所示新的权限将外部储存空间分成三个部分

* 特定目录(App-specific)，使用`getExternalFilesDir()`和`getExternalCacheDir()`方法访问。无需权限，且应用卸载时会自动删除
  * `getExternalFilesDir()`，一般用于存储文档，包括下载的安装包这类的文件。实际目录为SDCard/Android/data/你的应用的包名/files/
  * `getExternalCacheDir()`，一般用于缓存类文件，包括图片缓存，数据缓存等。实际目录为SDCard/Android/data/你的应用包名/cache/
  * 应用在卸载后，会将`App-specific`目录下的数据删除，如果在`AndroidManifest.xml`中声明：`android:hasFragileUserData="true"`用户可以选择是否保留。
* 照片、视频、音频这类媒体文件。使用`MediaStore`访问，当前应用下载媒体文件或编辑/访问当前应用下载的媒体文件不需要申请权限，访问其他应用的媒体文件时需要`READ_EXTERNAL_STORAGE`权限。
* 其他目录，使用[存储访问框架SAF](https://developer.android.google.cn/guide/topics/providers/document-provider?hl=zh_cn)（Storage Access Framwork）



如何适配

在android11之前可以在`AndroidManifest.xml`中添加 `android:requestLegacyExternalStorage="true"`来请求使用旧的存储模式。但是这个方案只能临时过渡使用，不推荐长期使用。

如果你已经适配Android 10，这里有个现象要**注意一下**：

如果应用通过升级安装，那么还会使用以前的储存模式（Legacy View）。只有通过首次安装或是卸载重新安装才能启用新模式（Filtered View）

故适配时，可以进入如下判断是否需要进行适配

````java
// 使用Environment.isExternalStorageLegacy()来检查APP的运行模式
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q && !Environment.isExternalStorageLegacy()) {
	// 只有在用户升级之后将数据移动至特定目录，未升级还是使用原来的模式
}
````

* 对于应用中涉及的文件操作适配，只需要进行分类并修改一下文件路径即可

以前我们习惯使用`Environment.getExternalStorageDirectory()`方法，现在一般存储文档（包括下载安装包类文件）放在`getExternalFilesDir()`目录下面，一般缓存类文件（图片缓存，数据缓存等）放在`getExternalCacheDir()`目录下面。

存储媒体类型文件需要使用`MediaStore`（图片：`MediaStore.Images` ，视频：`MediaStore.Video`，音频：`MediaStore.Audio`）

代码如下

````java
public static Uri createImageUri(Context context) {
    ContentValues values = new ContentValues();
    // 需要指定文件信息时，非必须
    values.put(MediaStore.Images.Media.DESCRIPTION, "This is an image");
    values.put(MediaStore.Images.Media.DISPLAY_NAME, "Image.png");
    values.put(MediaStore.Images.Media.MIME_TYPE, "image/png");
    values.put(MediaStore.Images.Media.TITLE, "Image.png");
    values.put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/test");

    return context.getContentResolver().insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
}
````



* 对于媒体资源的访问：比如图片选择器这类的场景。无法直接使用File，而应使用Uri。否则报错如下：

`java.io.FileNotFoundException: open failed: EACCES (Permission denied)`。

一般选择图片后不论是用于展示还是用于上传都是使用file格式，现在返回的Uri格式，所以我将最终选择的文件又转存进了`getExternalFilesDir()`，主要代码如下

````java
File imgFile = this.getExternalFilesDir("image");
if (!imgFile.exists()){
    imgFile.mkdir();
}
try {
    File file = new File(imgFile.getAbsolutePath() + File.separator + System.currentTimeMillis() + ".jpg");
    // 使用openInputStream(uri)方法获取字节输入流
    InputStream fileInputStream = getContentResolver().openInputStream(uri);
    FileOutputStream fileOutputStream = new FileOutputStream(file);
    byte[] buffer = new byte[1024];
    int byteRead;
    while (-1 != (byteRead = fileInputStream.read(buffer))) {
        fileOutputStream.write(buffer, 0, byteRead);
    }
    fileInputStream.close();
    fileOutputStream.flush();
    fileOutputStream.close();
    // 可以直接返回图片新路径 file.getAbsolutePath()
} catch (Exception e) {
    e.printStackTrace();        
}
````



* 如果你要获取图片中的地理位置信息，需要申请`ACCESS_MEDIA_LOCATION`权限，并使用MediaStore.setRequireOriginal()获取。下面是官方的示例代码：

````java
Uri photoUri = Uri.withAppendedPath(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, cursor.getString(idColumnIndex));
final double[] latLong;

// 从ExifInterface类获取位置信息
photoUri = MediaStore.setRequireOriginal(photoUri);
InputStream stream = getContentResolver().openInputStream(photoUri);
if (stream != null) {
    ExifInterface exifInterface = new ExifInterface(stream);
    double[] returnedLatLong = exifInterface.getLatLong();

    // If lat/long is null, fall back to the coordinates (0, 0).
    latLong = returnedLatLong != null ? returnedLatLong : new double[2];

    // Don't reuse the stream associated with the instance of "ExifInterface".
    stream.close();
} else {
    // Failed to load the stream, so return the coordinates (0, 0).
    latLong = new double[2];
}
````



#### 2.权限变化

* 在后台运行时访问设备位置信息需要权限

Android10引入了`android.permission.ACCESS_BACKGROUND_LOCATION`。该权限允许应用程序在后台访问位置。如果请求此权限，则还必须请求`android.permission.ACCESS_FINE_LOCATION `或 `android.permission.ACCESS_COARSE_LOCATION`权限。只请求此权限无效果

在Android 10的设备上，如果你的应用的 targetSdkVersion < 29，则在请求ACCESS_FINE_LOCATION 或ACCESS_COARSE_LOCATION权限时，系统会自动同时请求ACCESS_BACKGROUND_LOCATION。在请求弹框中，选择“始终允许”表示同意后台获取位置信息，选择“仅在应用使用过程中允许”或"拒绝"选项表示拒绝授权。

如果你的应用的 targetSdkVersion >= 29，则请求ACCESS_FINE_LOCATION 或 ACCESS_COARSE_LOCATION权限表示在前台时拥有访问设备位置信息的权。在请求弹框中，选择“始终允许”表示前后台都可以获取位置信息，选择“仅在应用使用过程中允许”只表示拥有前台的权限。

* 一些电话、蓝牙和WLAN的API需要精确位置权限

下面列举了Android 10中必须具有 ACCESS_FINE_LOCATION 权限才能使用类和方法:

电话

* TelephonyManager
  * getCellLocation()
  * getAllCellInfo()
  * requestNetworkScan()
  * requestCellInfoUpdate()
  * getAvailableNetworks()
  * getServiceState()
* TelephonyScanManager
  * requestNetworkScan()
* TelephonyScanManager.NetworkScanCallback
  * onResults()
* PhoneStateListener
  * onCellLocationChanged()
  * onCellInfoChanged()
  * onServiceStateChanged()

WLAN

* WifiManager
  * startScan()
  * getScanResults()
  * getConnectionInfo()
  * getConfiguredNetworks()
* WifiAwareManager
* WifiP2pManager
* WifiRttManager

蓝牙

* BluetoothAdapter
  * startDiscovery()
  * startLeScan()
* BluetoothAdapter.LeScanCallback
* BluetoothLeScanner
  * startScan()

我们可以根据上面提供的具体类和方法，在适配项目中检查是否有使用到并及时处理。



#### 3. 后台启动Activity的限制

简单解释就是**应用处于后台时，无法启动Activity**。比如点开一个应用会进入广告页，一般会有几秒的延时再跳转至首页。如果这期间你退到后台，那么你将无法看到跳转过程。而在之前的版本中，会强制弹出首页至前台。这样就会导致一个问题，点击推送消息时有时会无法正常跳转，针对这样的问题可以采取`pendingIntent`的方式，发送通知时使用`setContentIntent`方法。也可以通过申请各大手机厂商的应用权限或白名单。



#### 4. 深色主题

* 手动适配

其实适配的方法很简单，类似屏幕适配、国际化的操作，并不需要继承上面的主题。比如你要修改颜色，就在res 下新建 values-night目录，创建对应的colors.xml文件。将具体要修改的色值定义在里面。图标之类的也是一个思路，创建对应的 drawable-night目录。

只要你之前的代码不是硬编码且代码规范，那么适配起来还是很轻松。

* 自动适配（Force Dark）

如果您的应用采用浅色主题背景，则 Force Dark 会分析应用的每个视图，并在相应视图在屏幕上显示之前，自动应用深色主题背景。

应用必须选择启用 Force Dark，方法是在其主题背景中设置 `android:forceDarkAllowed="true"`或 `setForceDarkAllowed(boolean)` 在特定视图上控制 Force Dark。

使用`Force Dark`需要注意几点：

* 如果使用的是 `DayNight` 或 `Dark Theme` 主题，则设置`forceDarkAllowed` 不生效。
* 如果有需要排除适配的部分，可以在对应的View上设置`forceDarkAllowed`为false。

整体感受：**整体还是不错的，设置的色值会自动取反。但也因此颜色不受控制，能否达到预期效果是个需要注意的问题。追求快速适配可以采取此方案。**

**手动切换主题**

使用 `AppCompatDelegate.setDefaultNightMode(@NightMode int mode)`方法，其中参数`mode`有以下几种：

* 浅色 - `MODE_NIGHT_NO`
* 深色 - `MODE_NIGHT_YES`
* 省电模式 - `MODE_NIGHT_AUTO_BATTERY`
* 系统默认 - `MODE_NIGHT_FOLLOW_SYSTEM`

````java
public class ThemeHelper {
    public static final String LIGHT_MODE = "light";
    public static final String DARK_MODE = "dark";
    public static final String DEFAULT_MODE = "default";

    public static void applyTheme(@NonNull String themePref) {
        switch (themePref) {
            case LIGHT_MODE: { //浅色
                AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_NO);
                break;
            }
            case DARK_MODE: { //深色
                AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_YES);
                break;
            }
            default: {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                    //系统默认
                    AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_FOLLOW_SYSTEM);
                } else {
                    //省电模式
                    AppCompatDelegate.setDefaultNightMode(AppCompatDelegate.MODE_NIGHT_AUTO_BATTERY);
                }
                break;
            }
        }
    }
}
````



**监听深色主题切换**

首先在清单文件中给对应的Activity配置 `android:configChanges="uiMode"`：

````java
<activity
    android:name=".MyActivity"
    android:configChanges="uiMode" />
````

这样在`onConfigurationChanged`方法中就可以获取：

````java
@Override
public void onConfigurationChanged(@NonNull Configuration newConfig) {
    super.onConfigurationChanged(newConfig);
    int currentNightMode = newConfig.uiMode & Configuration.UI_MODE_NIGHT_MASK;
    switch (currentNightMode) {
        case Configuration.UI_MODE_NIGHT_NO:
            // 关闭
            break;
        case Configuration.UI_MODE_NIGHT_YES:
            // 开启
            break;
        default:
            break;    
    }
}
````

判断是否开启深色主题模式

````java
public static boolean isNightMode(Context context) {
    int currentNightMode = context.getResources().getConfiguration().uiMode & Configuration.UI_MODE_NIGHT_MASK;
    return currentNightMode == Configuration.UI_MODE_NIGHT_YES;
}
````



#### 5.标识符和数据

对不可重置的设备标识符实施了限制

受影响的方法包括：

Build

* [getSerial()](https://developer.android.google.cn/reference/android/os/Build#getSerial())

TelephonyManager

* [getImei()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getImei(int))
* [getDeviceId()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getDeviceId(int))
* [getMeid()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getMeid(int))
* [getSimSerialNumber()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getSimSerialNumber())
* [getSubscriberId()](https://developer.android.google.cn/reference/android/telephony/TelephonyManager#getSubscriberId())



### Android 9







### Android 8







### Android 7







### Android 6









#### Android 之前所有版本







