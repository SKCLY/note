## Kotlin详解



[全民Kotlin：Java我们不一样 ](https://mp.weixin.qq.com/s/FqXLNz5p9M-5vcMUkxJyFQ)

[全民Kotlin：你没有玩过的全新玩法](https://www.jianshu.com/p/884ca0a49e5e)

[kotlin-android-extensions插件也被废弃](https://blog.csdn.net/guolin_blog/article/details/113089706)



#### Kotlin比java优化的地方

**变量**

静态常量使用val修饰符，一般常量使用var修饰符。如果赋值是明确类型的值，可以不写明类型，如果需要明确类型使用`:`加数据类型跟在变量名后面。

**转换符**

Java中一般使用`String.format`，kotlin中直接在字符串中加入`${}`就可以直接引用外变定义变量，非常方便。

**类创建**

Java中需要写完类所有变量之后，再创建对应变量的`get`和`set`方法。

kotlin中只需要在变量定义后面跟上`get()`和`set(value)`方法，获取变量值是`field`。

更加简洁的写法，`class 类名()`，在大括号里面增加变量定义就可以了。

kotlin针对只接收数据的类还推出了一个数据类，这个类功能只能保存数据，写法是在`class`前面增加`data`关键字，内部变量就和函数传参一样的写法，编译器会自动生成`get`和`set`方法，可以节省大量代码。

**集合**

kotlin提供了基本集合类型：set，list和map，每种集合有两个类型：

* 一个是只读接口，只提供访问集合的操作，使用`listOf()`创建集合
* 一个可变接口，提供添加，删除和更新操作，使用`mutableListOf()`创建集合

`List`对应Java的`ArrayList`，`Set`对应Java的`LinkedHashSet`，`map`对应Java的`LinkedHashMap`。

还提供了很多操作集合的方法

* 空集合，emptyList()、emptySet() 与 emptyMap()。
* 添加，删除。add()，addAll()，remove()，removeAll()
* 过滤。filter()
* 映射。map()
* 将集合转换为字符串的函数。joinToString()
* 取集合的部分。Slice()，subList()

**创建对象**

java写法需要通过new来创建新变量

````java
Intent intent = new Intent();
````

kotlin简化创建变量不需要new

```kotlin
var intent = Intent()
```

**Unit**

`Unit`本身是一个用`Object`表示的单例，类型于Java中的`Void`，由于在Kotlin中一切方法/函数都是表达式，都必有一个返回值，如果没有`return`明确的指定返回值，一般都会自动加上`Unit`。

**Any**

`Any`其实跟Java里的`Object`是一样的，在Kotlin中`Any`取代了Java中的`Object`，是所有类的父类。

**Lambda**

kotlin中大量使用了Lambda表达式进行代码简化

**空安全**

在Java中不用强制处理空对象，所以经常会导致空指针出错，在Kotlin中对空对象进行限定，必需在编译时处理对象是否为空的情况，不然编译不通过。

当某个变量的值可以为 null 的时候，必须在声明处的类型后添加 ? 来标识该引⽤可为空，在使用这个变量只需要在变量名后面加上?来使用，当变量为null时结果为null，当变量不为null时结果会变量值。可以简化大量的判空处理代码。

如果确定变量为非空，可以使用⾮空断⾔运算符（!!）来表示变量一定有值。这样不推荐使用容易引起空指针错误。

**方法支持添加默认参数**

在java上我们可能会为了扩展某个方法进行多次重载。

```java
public void toast(String text) {
    toast(this, text, Toast.LENGTH_SHORT);
}

public void toast(Context context, String text) {
    toast(context, text, Toast.LENGTH_SHORT);
}

public void toast(Context context, String text, int time) {
    Toast.makeText(context, text, time).show();
}
```

在kotlin上面，无需要重载，可以直接在方法上面定义参数默认值，如果参数没有传参时就会使用默认值。

**无需findViewById**

`kotlin-android-extensions`插件可以直接使用xml中控件的id在java代码中引用到对应控件，不需要再通过`findViewById`来获取xml中的控件。



然后在Android Studio 4.1之后新建项目不会再自动引入`kotlin-android-extensions`插件，因为Google不再推荐使用，而是希望使用`ViewBinding`来代替`kotlin-android-extensions`。

`kotlin-android-extensions`插件的实现原理会帮我们生成一个`_$_findCachedViewById()`函数，在这个函数中会先尝试从一个HashMap中获取id所对应的控件实例，如果还没有缓存的话就调用`findViewByid()`去查找实例，并写入HashMap缓存中，其内部实现比较简单。

这样也暴露了一些问题

* 每个Activity都需要维护一个额外的HashMap来存储所有控件的实例，增加了内存开支
* HashMap读取速度是没有直接访问控件实例来得快，降低了程序的运行效率
* 实现对于开发者是黑盒，在一些情况下会出现开发者不可预知的问题，例如在RecyclerView的adapter中使用



**什么是ViewBinding**

ViewBinding总体来说其实非常简单，它的目的只有一个，就是为了避免编写findViewById，与DataBinding相比简单的多。

使用ViewBinding只需要二件事情，第一确保Android Sutdio版本3.6或更高的版本；第二在项目的build.gradle加入如下配置

````gradle
android {
	buildFeatuers {
		viewBinding true
	}
}
````

这样就可以在Activity，Fragment，Adapter，手动引入布局这4个方面使用ViewBinding了。

**在Activity中使用**

一旦启动ViewBinding的功能之后，Android Studio会自动为我们编写的布局文件生成一个Binding类，该类的命名规则是将布局文件名字按驼峰方式重命名后，再加上Binding作为结尾。比如我们定义一个activity_main.xml原布局，那它对应的Binding类就是ActivityMainBinding。

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.textView.text = "Hello"
    }
}
```

使用步骤如下

* 调用布局文件对应的Binding类的inflate()函数去加载布局，inflate()函数接收一个LayoutInfalter参数，在Activity中可以直接获取到。
* 调用Binding类的getRoot()函数可以得到布局文件根元素的实例，需要在setContentView()函数中传入，这要就能展示布局。
* 然后通过调用Binding类的getText()函数可以获取id为text的元素实例进行操作。

**在Fragment中使用**

在onCreateView中调用Binding类的inflate()函数去加载布局文件，其他都使用就和Activity一样。

**在Adapter中使用**

在onCreateViewHolder中调用Binding类的inflate()函数加载布局文件，返回ViewHolder并接收Binding类，这样在ViewHolder()直接通过布局Binding类就可以获取对应用控件实例。

```kotlin
class FruitAdapter(val fruitList: List<Fruit>) : RecyclerView.Adapter<FruitAdapter.ViewHolder>() {

    inner class ViewHolder(binding: FruitItemBinding) : RecyclerView.ViewHolder(binding.root) {
        val fruitImage: ImageView = binding.fruitImage
        val fruitName: TextView = binding.fruitName
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val binding = FruitItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return ViewHolder(binding)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val fruit = fruitList[position]
        holder.fruitImage.setImageResource(fruit.imageId)
        holder.fruitName.text = fruit.name
    }

    override fun getItemCount() = fruitList.size

}
```

**在引入布局中使用**

在布局中通过include引入布局，需要增加id才能通过ViewBinding获取引入的布局。

如果通过include引入的布局最外层是merge标签，那么不需要增加id，会生成一个引入xml文件对应的Binding类，来获取引入布局。



#### 作用域函数

作用域函数是为了方便对一个对象进行访问和操作，你可以对它进行空检查或者修改它的属性或者直接返回它的值等操作，其中有 let、with、run、apply、also 五个函数。

**let**

在let作用域内可以通过it指代该对象。返回值为作用域内的最后一行或指定return表达式，也就是作用域内的最后一条语句是非赋值语句，则默认情况下它是返回语句，否则返回一个Unit类型。

````kotlin
//一般写法
fun main() {
    val text = "Android轮子哥"
    println(text.length)
    val result = 1000
    println(result)
}
````

```kotlin
//let写法简化
fun main() {
    val result = "Android轮子哥".let {
        println(it.length)
        1000
    }
    println(result)
}
```

使用场景

对象进行空检查并访问或修改其属性。在android中从网络请求返回对象中取数据可以使用let



**with**

将某个对象作为参数，在作用域内通过this指代该对象。返回值函数块最后一行或指定return表达式。

```kotlin
//一般写法
fun main() {
    var person = Person("Android轮子哥", 100)
    println(person.name + person.age)
    var result = 1000
    println(result)
}
```

```kotlin
//with写法
fun main() {
    var result = with(Person("Android轮子哥", 100)) {
        println(name + age)
        1000
    }
    println(result)
}
```

使用场景

如果是非null的对象并且作用域中不需要返回值时。在android中经常在`RecyvlerView`中的`onBinderViewHolder`中`model`的属性映射到UI上使用。



**run**

run函数基本上和with函数是一样的。区别在于run不需要将对象做为参数，也弥补了with函数传入对象判空的问题，可以在外面做判空处理。返回值函数块最后一行或指定return表达式。

```Kotlin
//with写法
fun main(args: Array<String>) {
    val book: Book? = null
    with(book){
        this?.name = "《计算机网络》"
        this?.price = 40
    }
    print(book)
}
```

```kotlin
//run写法
fun main(args: Array<String>) {
    val book: Book? = null
    book?.run{
        name = "《计算机网络》"
        price = 40
    }
    print(book)
}
```

使用场景

适用于let和with的所有使用场景。



**apply**

apply函数和run函数很像，唯一不同点就是它们返回值不同，run函数是返回作用域最后一行的值，而apply函数返回是传入对象本身。

```kotlin
//一般写法
val person = Person("Android轮子哥", 100)
person.name = "HJQ"
person.age = 50
```

```kotlin
//apply写法
val person = Person("Android轮子哥", 100).apply {
    name = "HJQ"
    age = 50
}
```

使用场景

用于初始化对象或更改对象属性。一般android中使用场景：1.对象初始化。2.动态inflate出一个XML的view时并绑定数据。 3.对象多层判空处理。



**also**

also函数实际上和let函数很像，唯一的区别是返回值不一样，let函数是以闭包形式返回最后一行的值，而also函数返回的则是传入对象本身。

```kotlin
//let写法
fun main() {
    val result = "Android轮子哥".let {
        println(it.length)
        1000
    }
    println(result) // 打印：1000
}
```

```kotlin
//also写法
fun main() {
    val result = "Android轮子哥".also {
        println(it.length)
        1000
    }
    println(result) // 打印：Android轮子哥
}
```

使用场景

将数据指派给接收对象的属性之前验证对象。



#### 协程

[协程到底是怎么切换线程的?](https://juejin.cn/post/6981056016897015838#heading-7)

协程就是kotlin中的一个线程开发框架，它最强大的地方在于使得线程切换相比于android的thread和AsyncTask更加方便。

```kotlin
GlobalScope.launch(Dispatchers.Main) {
    //在主线程获取图片并设置在view中
    val image = suspendGetImage("id")
    avatarIv.setImage(image)
}
suspend fun suspendGetImage(imageId: String) {
    //切换成io线程去请求图片
    //图片请求成功之后返回主线程
    withContext(Dispatchers.IO) {
        getImageId(imageId)
    }
}
```

**协程的挂起是什么**

可以自动切换线程的切线程。

**挂起的非阻塞式是什么**

协程可以用看来阻塞的代码实现非阻塞的功能。



**kotlin中创建协程的有哪些方法**

[kotlin中创建协程的方法](https://blog.csdn.net/zhong_zihao/article/details/105145206)

* 可在全局创建协程的： lauch 与 runBlocking。lauch 与 runBlocking都能在全局开启一个协程，但 lauch 是非阻塞的 而 runBlocking 是阻塞的
  * 通过launch开启一个协程，协程体里的任务时就会先挂起（suspend），让launch后面的代码继续执行，直到协程体内的方法执行完成再自动切回来所在的上下文回调结果
  * 通过runBlocking开启一个协程，程序会等待执行完成runBlocking内的协程体代码，再执行它后面的代码。  runBlocking里的任务如果是非常耗时的操作时，会一直阻塞当前线程，在实际开发中很少会用到runBlocking。
* 可返回结果的协程：withContext 与 async 
  * 多个 withContext 任务是串行的， 且withContext 可直接返回耗时任务的结果。 
  * 多个`async`任务是并行的，`async`返回的是一个`Deferred<T>`，需要调用其`await()`方法获取结果。

