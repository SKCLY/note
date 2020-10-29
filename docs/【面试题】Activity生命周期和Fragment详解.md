# Activity和Fragment详解



### Activity之间，Fragment之间，Activity和Fragment如何通信？

1. 使用Bundle实现通信。局限：传输Bundle支持的数据类型，大小最大为1M。
2. Broadcast或者EventBus发送广播实现通信。局限：全局通知，其他页面和其他APP都会收到通知，key必需要全部唯一，不然会出现多处生效，引起不可预料的问题。
3. 使用数据存储通信，比如数据库，SharePreference，文件共享。局限：需要频繁的进行IO操作，操作比较复杂，而且使用完成之后需要删除(一般传递的不是需要长久保存的数据)。
4. 借助类的静态变量或者全局变量Application实现通信。局限：大量静态变量会减慢APP启动速度，增加APP占用内存。
5. 使用handler实现通信。handler定义为跨线程通信，使用场景不对，会造成代码冗余，不好维护。
6. livedataBus实现通信。优点：官方标准框架

```java
public class LiveDataBus {
    private Map<String, MutableLiveData<Object>> bus;
    private LiveDataBus() { bus = new HashMap<>(): }
    
	private static LiveDataBus liveDataBus = new LiveDataBus();
    public static getInstance { return liveDataBus }
    public synchronized <T> MutableLiveData<Object> with(String key, Class<T> type) {
        if (!bus.containKey(key)) {
            bus.put(key, new MutableLiveData<>());
        }
        return (MutableLiveData<T>)bus.get(key);
    }
}
```



#### Fragment和Activity之间的关系？

* Fragment是依赖于Activity的，不能独立存在，Activity是Fragment的一个容器。
* 一个Activity可以有多个Fragment，且可以动态添加或删除
* 一个Fragment可以被多个Activity使用
* Fragment有独立生命周期，能接收输入事件

Fragment能解决的问题：

* 模块化，可以不必把所有代码都写在Activity中，而是把Activity不同内容使用Fragment分离，从而增加灵活性
* 可重用：多个Activity可以重用一个Fragment
* 可适配：可以根据不同屏幕尺寸，屏幕方向，方便实现不同的布局

