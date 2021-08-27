

# 【面试题】高级UI相关

#### 1. LayoutParams是什么？ 

LayoutParams翻译过来就是布局参数，子View通过LayoutParams告诉父容器（ViewGroup）应该如何放置自己。故LayoutParams与ViewGroup是息息相关的，每个ViewGroup的子类都有自己对应的LayoutParams类，典型的如LinearLayout.LayoutParams和 FrameLayout.LayoutParams等。

#### 2. LayoutParams与MeasureSpec有什么关系？

View的MeasureSpec是由父容器的MeasureSpec和View自身的LayoutParams来共同决定的。

#### 3. MeasureSpec是什么？

MeasureSpec是View中的内部类，基本都是二进制运算。由于int是32位的，用高两位表示SpecMode，低30位表示SpecSize，MODE_SHIFT = 30的作用是移位。

SpecSize是指在某种SpecMode下的参考尺寸，其中SpecMode有如下三种：

* EXACTLY 父控件已经知道你所需的精确大小。 int EXACTLY = 1
* AT_MOST 子View大小不能大于父控件给你指定的size，但具体是多少看你实际的实现。 int AT_MOST = 2
* UNSPECIFIED 父控件不对你有任何限制。这种情况一般用于系统内部， 表示一种测量状态。（这个模式主要用于系统内部多次Measure的情形，并不是真的说你想要多大最后就真有多大） int UNSPECIFIED = 0

#### 4. MeasureSpce计算规则

getChildMeasureSpec()函数内部实现算法

![image](..\images\MeasureSpec计算规则.png)

对于不同的父容器和view本身不同的LayoutParams，view就可以有多种MeasureSpec。 

1. 当view采用固定宽高的时候(dp/px)，不管父容器的MeasureSpec是什么，view的MeasureSpec都是精确模式(EXACTLY)并且其大小遵循 Layoutparams中的大小； 
2. 当view的宽高是match_parent时，这个时候如果父容器的模式是精准模式(EXACTLY)，那么 view也是精准模式并(EXACTLY)且其大小是父容器的剩余空间，如果父容器是最大模式(AT_MOST)，那么view也是最大模式(AT_MOST)并且其大 小不会超过父容器的剩余空间； 
3. 当view的宽高是wrap_content时，不管父容器的模式是精准(EXACTLY)还是最大化(AT_MOST)， view的模式总是最大化(AT_MOST)并且大小不能超过父容器的剩余空间。 
4. Unspecified模式，这个模式主要用于系统内部多次measure的情况下，一般来说我们不需要关注此模式(这里注意自定义View放到ScrollView的情况需要处理).

#### 5. MeasureSpec有什么意义

SpecMode有UNSPECIFIED ，EXACTLY，AT_MOST三种情况只需要2位，SpecSize最大不会超过10000象素，故30位足够表示长度。

通过将 SpecMode 和 SpecSize 打包成一个 int 值可以避免过多的对象内存分配，为了方便操作，其提供了打包/解包方法

#### 6. 为什么需要measure?

无论是measure过程、layout过程还是draw过程，永远都是从View树的根节点开始测量或计算（即从
树的顶端开始），一层一层、一个分支一个分支地进行（即树形递归），最终计算整个View树中各个View，最终确
定整个View树的相关属性。

![image](..\images\View层级结构.png)

#### 7. Android两种坐标系

![image](..\images\Android屏幕坐标系.png)

![image](..\images\Android视图坐标系.png)

#### 8. getMeasureWidth和getWidth的区别

getMeasureWidth()：在measure()结束后就可以获取到对应的值，通过setMeasureDimension()方法来进行设置

getWidth()：在layout()过程结束后才能获取到，通过视图右边坐标减去左边坐标计算而来

