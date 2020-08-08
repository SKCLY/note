# 事件分发

### 单点触摸

![image](..\images\事件分发-单点触摸.png)

### 多点触摸

![image](..\images\事件分发-多点触摸.png)



### View的体系结构

![image](..\images\View的体系结构.png)

### View事件分发大概流程

![image](..\images\事件分发大流程.png)

### 事件处理函数

![image](..\images\事件处理函数.png)

为什么View会有dispatchTouchEvent方法？

View可以注册很多事件监听，例如单击事件(onClick)，长按事件(onLongClick)，触摸事件(onTouch)，并且View本身也有onTouchEvent方法，这么多事件相关管理方法就是dispatchTouchEvent，主要是用来分发View自身的事件。

与View事件相关的各个方法调用顺序与其原因？

![image](..\images\View事件分发调用顺序.png)

* 单击事件(onClickListener)需要两个事件(ACTION_DOWN和ACTION_UP)才能触发，当OnClick判断结束时用户手指已经离开屏幕，那其他事件都无法触发，故应该最后分发。
* 长按事件(onLongClickListener)同理，需要长时间等待才能判断触发，因不需要ACTION_UP，放在onClick前面
* 触摸事件(onTouchListener)，如果用户注册触摸事件，说明用户要自己处理触摸，故放在最前面
* View的onTouchEvent提供了一种默认的处理方式，如果用户不想自己处理触摸事件可以使用默认处理方式，故放在onTouchListener后面

#### View的dispatchTouchEvent事件分发源码分析

```java
// 判断OnTouchListener#onTouch返回结果
// 返回为true则result为true
if (li != null && li.mOnTouchListener != null
	&& (mViewFlags & ENABLED_MASK) == ENABLED
	&& li.mOnTouchListener.onTouch(this, event)) {
		result = true;
}

// onTouchEvent()内部处理了onClickListener和onLongClickListener
// 故当OnTouchListener#onTouch返回为true时，不会执行onClickListener和onLongClickListener方法
if (!result && onTouchEvent(event)) {
	result = true;
}

onTouchEvent方法
    case MotionEvent.ACTION_UP:
		// 处理点击事件
        if (!post(mPerformClick)) {
            performClickInternal();
        }

	case MotionEvent.ACTION_DOWN:
		// 长按500ms后触发onLongClickListener#onLongClick事件
        checkForLongClick(0, x, y);
```

### 事件分发源码分析

ACTION_DOWN事件分发传递过程

````java

````

ACTION_MOVE事件分发传递过程