## Dialog不能使用ApplicationContext原理

dialog的构造函数中获取WindowManager通过如下方式

````java
mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
````

 #### 1.正常流程

context为Activity#getSystemService()函数返回mWindowManager，mWindowManager是在Activity#attach()函数中调用Window#setWindowManager()里面通过如下代码创建，并通过Window#getWindowManager()函数返回。

````java
mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);

public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
    return new WindowManagerImpl(mContext, parentWindow);
}
````

调用WindowManagerImpl#createLocalWindowManager()时会把Activity#attach()中创建的phonewindow作为参数传递进来当作mParentWindow，故在WindowManagerGlobal#addView()里面会执行如下代码获取dialog的token值，

````java
if (parentWindow != null) {
    parentWindow.adjustLayoutParamsForSubWindow(wparams);
}
````

最后成功展示dialog。

#### 2.使用ApplicationContext

context.getSystemService最终的实现是在SystemServiceRegistry#getSystemService()里面，会直接调用如下代码来创建mWindowManager，故当前windowManager#mParentWindow为null。

````java
new WindowManagerImpl(ctx);
public WindowManagerImpl(Context context) {
    this(context, null);
}
private WindowManagerImpl(Context context, Window parentWindow) {
    mContext = context;
    mParentWindow = parentWindow;
}
````

其他流程和上面一样，故当前情况下dialog的token值为null，展示时会报错。