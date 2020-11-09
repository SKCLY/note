# RecyclerView回收复用

#### ReceyclerVIew回收和复用的是什么？

回收复用的都是ViewHolder



#### 回收到哪里？从哪里获取复用？

mChangeScrap与mAttachedScrap用来缓存还在屏幕内的ViewHolder

mCachedViews用来缓存移除屏幕之外的ViewHolder

mViewCacheExtension用来给用户自定义扩展缓存，需要用户自己管理View的创建和缓存

RecycledViewPool是ViewHolder缓存池



#### RecyclerView如何实现局部刷新？

```java
// 刷新全部列表，会有刷新动画
mAdapter.notifyDataSetChanged();
// 刷新指定位置的item，会有刷新动画
mAdapter.notifyItemChanged(int position);
// 刷新指定位置的item
// payload需要刷新item的数据
mAdapter.notifyItemChanged(int position, Object payload)
// 从指定位置刷新指定个数item，会有刷新动画
// payload需要刷新item的数据
mAdapter.notifyItemRangeChanged(int positionStart, int itemCount, Object payload);
// 从指定位置插入一个item并刷新，需要修改数据源
mAdapter.notifyItemInserted(int position);
// 将item移动位置并刷新，需要修改数据源
mAdapter.notifyItemMoved(int fromPosition, int toPosition);
// 从指定位置删除一个item并刷新，需要修改数据源
mAdapter.notifyItemRemoved(int position);
```



#### 什么时候去回收？ 什么时候去复用？

在滑动事件开始进行回收和复用，具体实现是在tryGetViewHolderForPositionByDeadline方法中。

onlayout的时候也会进行复用。



#### 能大概讲一下RecyclerView回收复用规则？

在滑动事件开始进行回收和复用，具体实现是在tryGetViewHolderForPositionByDeadline方法中。

复用流程

代码逻辑：滑动 Move 事件 --> scrollByInternal --> scrollStep --> mLayout.scrollVerticallyBy 
--> scrollBy  --> fill --> layoutChunk  --> layoutState.next --> layoutState.next --> getViewForPosition --> tryGetViewHolderForPositionByDeadline -->

逻辑上分四种情况去获取ViewHolder

1. 通过postion从`mChangedScrap`获取，主要与动画相关（实现方法getChangedScrapViewForPosition）
2. 通过postion从`mAttachedScrap`，`mCachedViews`中获取ViewHolder（实现方法getScrapOrHiddenOrCachedHolderForPosition）
3. 通过ViewType和ItemId从`mAttachedScrap`，`mCachedViews`中获取ViewHolder（实现方法getScrapOrCachedViewForId）
4. 通过postion从`mViewCacheExtension`获取ViewHolder，用户自定义获取，用于局部刷新（实现mViewCacheExtension.getViewForPositionAndType）
5. 通过ViewType从`RecycledViewPool`获取ViewHolder（实现getRecycledViewPool().getRecycledView）
6. 如果还没有获取ViewHolder通过调用adapter.createViewHolder去创建ViewHolder
7. 调用onBindeViewHolder完成ViewHolder的数据绑定



回收的流程

回收代码流程

LinearLayoutManager.onLayoutChildren  -->  detachAndScrapAttachedViews  -->  scrapOrRecycleView  -->  scrapOrRecycleView方法

当ViewHolder数据无效的，并且数据没有部分移除更新，执行如下流程

recycler.recycleViewHolderInternal(viewHolder);  处理`mCachedViews `和 `RecycledViewPool`回收

1. 当ViewHolder对象没有发现变化时去加入`mCachedViews`缓存中（有些列表可以根据ViewType展示多个ViewHolder）

2. 当`mCachedViews`缓存列表存满时，把`mCachedViews`的第一个ViewHolder缓存到`RecycledViewPool`里面，并从`mCachedViews`缓存中移除
   * 故`mCachedViews`是先进先出策略，默认大小为2
3. 当ViewHolder对象发生变化时，直接把ViewHolder缓存到`RecycledViewPool`里面（具体实现addViewHolderToRecycledViewPool）
   * 当`RecycledViewPool`存储满了，后面的ViewHolder就不在存储，默认大小为5



当ViewHolder数据有效，或数据有部分移除更新

recycler.scrapView(view);

当ViewHolder标记为`remove`和`invalde`，或者完全没有变化时，会缓存到`mAttachedScrap`当中

当ViewHolder标记为`updated`（执行adapter的nofityItemChanged时）时会缓存到`mChangedScrap`当中

所以当复用ViewHolder时先从这两个列表中获取数据，这些数据是已经在屏幕当中的

