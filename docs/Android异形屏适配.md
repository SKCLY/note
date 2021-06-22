## Android异形屏适配

异形屏适配主要需要解决的两个问题

* 适应更长的屏幕
* 防止页面内容被刘海遮挡

第一点解决方案是声明最大长宽比。

第二点，如果应用或者页面本身不需要全屏或者使用沉浸式状态栏，是不需要适配。



#### 声明最大长宽比

以前的普通屏长宽比为16:9，全面屏手机的屏幕长宽比增大了很多，出现了很多18:9的手机，如果不适配的话就会页面过短，手机屏幕底部会出现一块未利用区域

解决方案就是在`AndroidManifest.xml`中添加最大长宽比

```xml
<application>
	<meta-data
        android:name="android.max_aspect"
        android:value="2.4" />
</application>
```

设置之后在Android8.0及更高版本中，会根据页面布局填充整个屏幕

在Android7.0及更低版本中，则系统会将应用界面的大小限制为宽高比为 16:9 的窗口。如果应用在具有较大屏幕宽高比的设备上运行，则该应用会以一个 16:9 的宽屏显示（上下各留出一部分屏幕不用）。



#### 适配异形屏

**Android9.0及更高版本适配**

Google官方推出出全新的`DisplayCutout`类来确定非功能区域的位置和形状，这些区域不应显示内容，从而来完全适配。

DisplayCutout主要有如下几个方法

主要用于获取凹口位置和安全区域的位置等。主要接口如下所示：

| 方法                 | 接口说明                                                     |
| -------------------- | ------------------------------------------------------------ |
| getBoundingRects()   | 返回Rects的列表，每个Rects都是显示屏上非功能区域的边界矩形。 |
| getSafeInsetLeft ()  | 返回安全区域距离屏幕左边的距离，单位是px。                   |
| getSafeInsetRight () | 返回安全区域距离屏幕右边的距离，单位是px。                   |
| getSafeInsetTop ()   | 返回安全区域距离屏幕顶部的距离，单位是px。                   |
| getSafeInsetBottom() | 返回安全区域距离屏幕底部的距离，单位是px。                   |

主要实现代码如下

````java
public class NotchActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getNotchParams();
    }

    @TargetApi(28)
    public void getNotchParams() {
        final View decorView = getWindow().getDecorView();

        decorView.post(new Runnable() {
            @Override
            public void run() {
                DisplayCutout displayCutout = decorView.getRootWindowInsets().getDisplayCutout();
                Log.e("TAG", "安全区域距离屏幕左边的距离 SafeInsetLeft:" + displayCutout.getSafeInsetLeft());
                Log.e("TAG", "安全区域距离屏幕右部的距离 SafeInsetRight:" + displayCutout.getSafeInsetRight());
                Log.e("TAG", "安全区域距离屏幕顶部的距离 SafeInsetTop:" + displayCutout.getSafeInsetTop());
                Log.e("TAG", "安全区域距离屏幕底部的距离 SafeInsetBottom:" + displayCutout.getSafeInsetBottom());
                
                List<Rect> rects = displayCutout.getBoundingRects();
                if (rects == null || rects.size() == 0) {
                    Log.e("TAG", "不是刘海屏");
                } else {
                    Log.e("TAG", "刘海屏数量:" + rects.size());
                    for (Rect rect : rects) {
                        Log.e("TAG", "刘海屏区域：" + rect);
                    }
                }
            }
        });
    }
}
````



**异形屏显示模式**

````java
WindowManager.LayoutParams lp = getWindow().getAttributes();
lp.layoutInDisplayCutoutMode = WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_ALWAYS;
getWindow().setAttributes(lp);
````

Android P中新增了一个布局参数属性`layoutInDisplayCutoutMode`，包含了三种不同的模式，如下所示：

| 模式                                      | 模式说明                                                     |
| ----------------------------------------- | ------------------------------------------------------------ |
| LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT     | 只有当DisplayCutout完全包含在系统栏中时，才允许窗口延伸到DisplayCutout区域。 否则(全屏模式下)窗口布局不与DisplayCutout区域重叠。 |
| LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER       | 该窗口决不允许与DisplayCutout区域重叠。                      |
| LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES | 该窗口始终允许延伸到屏幕短边上的DisplayCutout区域。          |

**总结**

如果页面存在状态栏

* 不用适配，因为刘海区域会包含在状态栏中了。
* 如果不想看到刘海区域，可以使用`LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER`将刘海区域变成一条黑色边。

页面全屏展示

* 不适配的话刘海区域会出现一条黑色边。
* 要做到真正全屏的话，那么就先要获取到刘海的区域（危险区域），内容部分（操作按钮等）应当避开危险区域，保证在安全区域中展示。也需要设计的配合，个人感觉第一条就可以



#### Android8之前的刘海适配 

[项目链接](https://github.com/smarxpan/NotchScreenTool)

#### 华为

**使用刘海区域**

使用新增的`meta-data`属性`android.notch_support`。 在应用的`AndroidManifest.xml`中增加`meta-data`属性，此属性不仅可以针对`Application`生效，也可以对`Activity`配置生效。 如下所示：

````xml
<meta-data android:name="android.notch_support" android:value="true"/>
````

默认情况下华为异形屏手机会对页面进行处理，竖屏场景的特殊下移或者是横屏场景的右移特殊处理，用来防止页面被刘海遮挡无法显示。该属性可以去除这种操作，将页面全屏显示但也会部分页面被遮挡。

**是否有刘海屏**

通过以下代码即可知道华为手机上是否有刘海屏了，`true`为有刘海，`false`则没有。

````java
public static boolean hasNotchAtHuawei(Context context) {
    boolean ret = false;
    try {
        ClassLoader classLoader = context.getClassLoader();
        Class HwNotchSizeUtil = classLoader.loadClass("com.huawei.android.util.HwNotchSizeUtil");
        Method get = HwNotchSizeUtil.getMethod("hasNotchInScreen");
        ret = (boolean) get.invoke(HwNotchSizeUtil);
    } catch (ClassNotFoundException e) {
        Log.e("Notch", "hasNotchAtHuawei ClassNotFoundException");
    } catch (NoSuchMethodException e) {
        Log.e("Notch", "hasNotchAtHuawei NoSuchMethodException");
    } catch (Exception e) {
        Log.e("Notch", "hasNotchAtHuawei Exception");
    } finally {
        return ret;
    }
}
````

**刘海尺寸**

华为提供了接口获取刘海的尺寸，如下：

````java
//获取刘海尺寸：width、height
//int[0]值为刘海宽度 int[1]值为刘海高度
public static int[] getNotchSizeAtHuawei(Context context) {
    int[] ret = new int[]{0, 0};
    try {
        ClassLoader cl = context.getClassLoader();
        Class HwNotchSizeUtil = cl.loadClass("com.huawei.android.util.HwNotchSizeUtil");
        Method get = HwNotchSizeUtil.getMethod("getNotchSize");
        ret = (int[]) get.invoke(HwNotchSizeUtil);
    } catch (ClassNotFoundException e) {
        Log.e("Notch", "getNotchSizeAtHuawei ClassNotFoundException");
    } catch (NoSuchMethodException e) {
        Log.e("Notch", "getNotchSizeAtHuawei NoSuchMethodException");
    } catch (Exception e) {
        Log.e("Notch", "getNotchSizeAtHuawei Exception");
    } finally {
        return ret;
    }
}
````

主要是通过反射`HwNotchSizeUtil`工具类来获取相应刘海信息，`hasNotchInScreen`判断是否为刘海屏，`getNotchSize`来获取刘海宽高属性。



#### VIVO

vivo在**设置**--**显示与亮度**--**第三方应用显示比例**中可以切换是否全屏显示还是安全区域显示。

**是否有刘海屏**

```java
public static final int VIVO_NOTCH = 0x00000020;//是否有刘海
public static final int VIVO_FILLET = 0x00000008;//是否有圆角

public static boolean hasNotchAtVoio(Context context) {
    boolean ret = false;
    try {
        ClassLoader classLoader = context.getClassLoader();
        Class FtFeature = classLoader.loadClass("android.util.FtFeature");
        Method method = FtFeature.getMethod("isFeatureSupport", int.class);
        ret = (boolean) method.invoke(FtFeature, VIVO_NOTCH);
    } catch (ClassNotFoundException e) {
        Log.e("Notch", "hasNotchAtVoio ClassNotFoundException");
    } catch (NoSuchMethodException e) {
        Log.e("Notch", "hasNotchAtVoio NoSuchMethodException");
    } catch (Exception e) {
        Log.e("Notch", "hasNotchAtVoio Exception");
    } finally {
        return ret;
    }
}
```

**刘海尺寸**

vivo不提供接口获取刘海尺寸，目前vivo的刘海宽为100dp,高为27dp。

![](C:\DevelopProject\learn\note\images\vivo异形屏适配.jpg)



#### OPPO

OPPO目前在设置 -- 显示 -- 应用全屏显示 -- 凹形区域显示控制，里面有关闭凹形区域开关。

**是否有刘海屏**

````java
public static boolean hasNotchInScreenAtOPPO(Context context) {
    return context.getPackageManager().hasSystemFeature("com.oppo.feature.screen.heteromorphism");
}
````

**刘海尺寸**

OPPO不提供接口获取刘海尺寸，目前其有刘海屏的机型尺寸规格都是统一的。不排除以后机型会有变化。 其显示屏宽度为1080px，高度为2280px。刘海区域则都是宽度为324px, 高度为80px。



#### 小米

**是否有刘海屏**

系统增加了 property `ro.miui.notch`，值为1时则是 Notch 屏手机。

```java
private static boolean isNotch() {
    try {
        Method getInt = Class.forName("android.os.SystemProperties").getMethod("getInt", String.class, int.class);
        int notch = (int) getInt.invoke(null, "ro.miui.notch", 0);
        return notch == 1;
    } catch (Throwable ignore) {}
    return false;
}
```

**刘海尺寸**

小米的状态栏高度会略高于刘海屏的高度，因此可以通过获取状态栏的高度来间接避开刘海屏，获取状态栏的高度代码如下：

```java
public static int getStatusBarHeight(Context context) {
    int statusBarHeight = 0;
    int resourceId = context.getResources().getIdentifier("status_bar_height", "dimen", "android");
    if (resourceId > 0) {
    	statusBarHeight = context.getResources().getDimensionPixelSize(resourceId);
    }
    return statusBarHeight;
}
```

