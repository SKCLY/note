# 性能优化-数据结构

各种数据结构存取速度对比。测试数据结构(主图，标题，副标题，详情，三个价格，6张详情图)

| 数据量 | ArrayList         | LinkedList          | HashMap           | SparseArray       | ArrayMap          |
| ------ | ----------------- | ------------------- | ----------------- | ----------------- | ----------------- |
| 10000  | 存 54ms  取 7ms   | 存 39ms  取 86ms    | 存 10ms  取 3ms   | 存 20ms  取 1ms   | 存 10ms  取 3ms   |
| 100000 | 存 137ms  取 20ms | 存 71ms  取 18860ms | 存 105ms  取 21ms | 存 169ms  取 12ms | 存 104ms  取 96ms |

#### ArrayList 

内部实现是数组

get获取速度比较快，数组内存连续，直接根据下标可以获取对应位置的值

add 和 remove速度比较慢，内部实现使用System.arraycopy，在列表中间增加或删除一个元素都需要将每个元素都移动一遍，故性能比较差。(只在数组最后add和remove速度和get一样快)

#### LinkedList

内部实现是链表

add和remove速度比较快

* 增加节点只需要在把最后节点的next指向新增节点，新增节点的pre指向最后一个节点就完成
* 删除节点只需要把删除节点的next和pre设置为null，把前节点的next指向后面节点，把后面节点的pre指向前面节点

查询节点速度比较慢。内部存储是无序的，需要遍历所有节点找出需要节点。

#### HashMap

包含了ArrayList和LinkedList全部优点

在java1.7之前hashmap结构是 数组 + 链表，在java1.8之后  数组 + 链表 + 红黑树

![image](..\images\hashmap结构.png)

存储对象逻辑：通过对key的hashCode()进行hash，并计算下标(位运算hash & length-1)，从而获得buckets的位置，当前位置为空直接将value存入，当前位置不为空(hash冲突)会使用链表来保存当前插入节点(需要轮询当前链表操作耗时)。
根据当前bucket的占用情况自动调整容量(加载因子默认为0.75f)，调整容量会原来的2倍，故有25%的空间浪费以空间换时间

#### hashMap计算下标为什么使用与运算，而不是真正的求模运算？

CPU所有的计算都是使用位运算进行，故使用与运算对CPU来说是最快的操作

#### hashMap什么时候需要扩容，扩容逻辑是什么？

根据当前bucket的占用情况自动调整容量(加载因子默认为0.75f)，调整容量会是原来的2倍，并将所有节点重新hash计算重新放置位置，扩容操作非常耗性能，使用hashMap要预置长度，尽量避免出现hashMap扩容(计算 length/0.75 + 1)。

hashmap默认长度为16，数组长度必需是2的次幂。为了减少哈希碰撞的概率，不是2次幂数在进行与运算时，更加容易出现哈希碰撞

#### 为什么hashMap加载因子默认为0.75f

增加hashMap长度会减少hash碰撞概率，故不能全部用完hashMap长度，但如果hashMap的长度过长，又会占用太多内存。

经过大量计算得到当加载因子在 0.6~0.75 区间效率最高

#### 什么是哈希碰撞？怎么解决？

多个对象的hashcode求模运算得到的结果相同就会产生哈希碰撞。

1. 通过扩容来解决用于减少hash碰撞概率

2. 通过链表法解决，把新加的节点加入到已经存储的节点后面，形成一个链表



#### SparseArray

SparseArray是android为优化HashMap的空间浪费而开发的数据结构，内部结构是双数组

![image](..\images\SparseArray结构.png)

特点：key(只能是int类型)和value分别用一个数组保存，key数组是按照从小到大排序的，key数组和value数组是一一对应的。

添加：使用两分法找到key对应key数组的位置，将key和value分别放入对应的数组相同的位置当中

扩容：将当前数组根据value的位置分成两个数组，把value放在第一个数组最后，最后拼成一个数组，需要使用两次arraycopy

获取：使用两分法查找key对应key数组的位置，在value数组相同位置获取value

删除：软删除，将当前位置的内容标记为DELETE，下次可以直接使用，减少数组操作

优点：

1. 不需要像HashMap一样需要25%的空间浪费，增加一个扩容一个
2. 使用二分查找int类型的key数组速度非常快
3. 软删除，将删除的内容标记为DELETE，而且下次使用只需要直接赋值不需要位置，越使用速度越快



#### ArrayMap [详情](http://gityuan.com/2019/01/13/arraymap)

优点了SparseArray的key值只能使用int的限制，可以和HashMap一样可以使用Object

![image](..\images\ArrayMap结构.png)

结构：ArrayMap也是使用了两个数组，一个存放key的hash值的有序整数数组，一个存入key/value对的Object数组

这样做的好处是，当新增加节点时，只需要增加hash数组的长度，而不需要像HashMap一样重建hash映射。

当ArrayMap删除数据时，会进行数组的缩减，进一步减少内存占用。

##### 扩容机制

容量扩张

1. 当map个数满足size<4时，扩容后大小为4
2. 当map个数满足4<=size<8时，扩容后大小为8
3. 当map个数满足size>=8时，则扩容后大小为原来1.5倍

容量缩减

只有当存储数据数量小于数组空间的1/3情况下进行容量缩减

1. 如果size<=8，则设置新大小为8
2. 如果size>8，则设置新大小为原大小的0.5倍

##### 缓存机制

缓存存储

在释放数组空间时，满足释放的大于等于4或8，且对应缓存池个数未达上限(上限为10个)，会把array加入到缓存池中。加入的方式是将数组array的第0个元素指向原有的缓存池，第1个元素指向hash数组地址，第2个元素之后全部清空，再把缓存池头部指向最新array[0]位置，缓存池大小加1。

![image](..\images\ArrayMap缓存机制.png)

缓存使用

当分配内存时，如果分配的大小等于4或者8，且缓存池不为空，则会从缓存池中取数组。取出方式是取出缓存池为array并赋值给mArray，将缓存池指向上一条缓存，将array的第1个元素赋值为mHashs，把mArray的第0个元素和第1个元素置为null，缓存池大小减1

![image](..\images\ArrayMap缓存机制1.png)

故使用ArrayMap时最好写成Array[8]。

#### 总结

1. ArrayMap比HashMap更加节省内存，当数据量不大(数据量小于1000)的情况下推荐使用ArrayMap
2. HashMap需要创建额外对象来保存每一个放入map的entry，且容量利用率比ArrayMap低
3. SparseArray比ArrayMap节省1/3的内存，但SparseArray的key只能为int类型，int类型Map推荐使用SparseArray。除了SparseArray还有 SparseLongArray，SparseBooleanArray两种。