## Fragment懒加载解析

#### 什么是Fragment懒加载

所谓的Fragment懒加载就是当Fragment可见的时候我们再去请求数据显示数据。



#### 为什么需要Fragment懒加载

fragment在这两种情况下会进行预加载，如下

* 使用ViewPager加载fragment，ViewPager内部的缓存机制为了防止在切换的时候出现卡顿的现象，提前将目标Fragment的左右Fragment都会加载好，但是预加载会造成不必要的流量损耗，而且当页面布局比较复杂缓存fragment过多，会造成初次进入页面加载卡顿。viewpager可以通过`setOffscreenPageLimit(int limit)`来设置预加载页面数量，默认为1
* 不使用ViewPager，一次性将多个Fragment加载好，通过hide和show方法控制Fragment的显示，这样也存在预加载的情况。



#### Fragment懒加载解决什么问题

* Fragment的懒加载就是解决Fragment配合ViewPager使用时的预加载，预加载会造成不必要的网络请求，这样会消耗用户的流量。
* 以前网速相对比较慢，页面布局也相对简单，预加载机制可以很好的提升页面流畅度。现在网速相对快，整体页面布局也更加复杂(有大量的图片和视频)，还使用预加载页面反而会超成页面卡顿。



**ViewPager懒加载**

我们观察到每次Fragment切换的时候都会将左右的Fragment加载好，同时生命周期方法执行到`onResume()`，但是当前由于使用ViewPager的缓存机制，Fragment执行到生命周期`onResume()`并不会显示，而是通过`setUserVisibleHint(boolean isVisibleToUser)`方法返回参数来表示当前Fragment是否真正可见，当Fragment是真正可见的时，`isVisibleToUser`为true，否则为false。

* 我们可以在onActivityCreated中通过isVisibleToUser来判断是否可以去请求数据

```java
//通过记录isVisibleToUser返回参数，true才需要加载数据
public void onActivityCreated(@Nullable Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    if (mIsVisibleToUser) {
        getData();
    }
}
```

* 这样看似解决了预加载的问题但是又会出现另外一个问题，因为已经被ViewPager缓存的Fragment的`onActivityCreated`方法已经执行了，这个时候我们进行fragment切换是不会加载数据了，我们可以在`setUserVisibleHint()`方法中加入请求数据的方法。由于`setUserVisibleHint`是在Fragment的生命周期之前执行的，需要判断当前Fragment的view是否加载完成，否则加载数据返回容易出现view空指针的问题。

```java
@Override
public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    mIsViewCreatetd = true;
}

@Override
public void setUserVisibleHint(boolean isVisibleToUser) {
    super.setUserVisibleHint(isVisibleToUser);
    mIsVisibleToUser = isVisibleToUser;
    if (mIsVisibleToUser && mIsViewCreatetd) {
        getData();
    }
}
```

* 完成上面逻辑，可以实现frament切换之后再去加载数据，但是如果切换回已经加载过数据的fragment还会重新去加载数据，如果是列表会刷新用户的浏览记录，一般刷新数据是给到用户手动下拉刷新的，故需要增加一个是否加载数据标识。

```java
//当前fragment是否可见
private boolean isVisibleToUser;
//当前fragment布局是否加载完成
private boolean isViewCreated;
//当前fragment是否加载过数据
private boolean isLoadData;

@Override
public void setUserVisibleHint(boolean isVisibleToUser) {
    super.setUserVisibleHint(isVisibleToUser);
    this.isVisibleToUser = isVisibleToUser;
    initData();
}

@Override
public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    isViewCreated = true;
}

@Override
public void onActivityCreated(@Nullable Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    initData();
}

private void initData() {
    if (mIsVisibleToUser && isViewCreated && !isLoadData) {
        getData();
        isLoadData = true;
    }
}

protected void getData() {}
```

总结实现懒加载如下

* 重写`setUserVisibleHint()`方法，根据`isVisibleToUser`判断当前Fragment是否处于隐藏
* 在`onViewCreated`中设置一个标记位，判断当前Fragment布局是否加载完成
* 每次调用完获取数据的方法，设置一个已请求数据的标志位
* viewpager的`setOffscreenPageLimit(int limit)`方法设置数量为所有fragment数量



**add+hide+show加载Fragment**

主要加载代码如下

```java
private void initFragment() {
    fragments.add(new FragmentOne());
    fragments.add(new FragmentTwo());
    fragments.add(new FragmentThree());
    fragments.add(new FragmentFour());
    fragments.add(new FragmentFive());
    loadFragments();
}

//加载所有的Fragment
private void loadFragments() {
    FragmentManager manager = getSupportFragmentManager();
    FragmentTransaction transaction = manager.beginTransaction();
    for (Fragment fragment : fragments) {
        transaction.add(R.id.mContainerView, fragment, fragment.getClass().getName());
        if (!(fragment instanceof FragmentOne)) {
            transaction.hide(fragment);
        }
    }
    transaction.commitAllowingStateLoss();
}

//点击Item切换不同的Fragment
private void initFragmentClick() {
    mBottomNavigationView.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
        @Override
        public boolean onNavigationItemSelected(@NonNull MenuItem menuItem) {
            switch (menuItem.getItemId()) {
                case R.id.home:
                    showFragment(fragments.get(0), fragments.get(selectedId));
                    selectedId = 0;
                    break;
                case R.id.wechat:
                    showFragment(fragments.get(1), fragments.get(selectedId));
                    selectedId = 1;
                    break;
                case R.id.project:
                    showFragment(fragments.get(2), fragments.get(selectedId));
                    selectedId = 2;
                    break;
                case R.id.system:
                    showFragment(fragments.get(3), fragments.get(selectedId));
                    selectedId = 3;
                    break;
                case R.id.setting:
                    showFragment(fragments.get(4), fragments.get(selectedId));
                    selectedId = 4;
                    break;
                default:
                    break;
            }
            return true;
        }
    });
}

//判断显示那个Fragment
private void showFragment(Fragment show, Fragment hide) {
    FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
    if (show != hide)
        transaction.hide(hide).show(show).commitAllowingStateLoss();
}
```

同样执行上面所有代码，初始化的时候就会所有fragment都会执行到`onActivityCreated()`，都会加载数据和页面。上面的代码是使用hide和show方法来控制Fragment的显示，使用这两个方法的时候不会去执行Fragment的生命周期方法，而是会执行`onHiddenChanged(boolean hidden)`方法，参数hidden表示当前的Fragment是否不可见 true/false。

这种方式的懒加载可以实现如下

* 重写`onHiddenChange()`方法，根据`hidden`判断当前Fragment是否处于隐藏
* 在`onViewCreated`中设置一个标记位，判断当前Fragment布局是否加载完成
* 每次调用完获取数据的方法，设置一个已请求数据的标志位
* viewpager的`setOffscreenPageLimit(int limit)`方法设置数量为所有fragment数量

实现代码如下

````java
//当前fragment是否可见
private boolean isHidden;
//当前fragment布局是否加载完成
private boolean isViewCreated;
//当前fragment是否加载过数据
private boolean isLoadData;

@Override
public void onActivityCreated(@Nullable Bundle savedInstanceState) {
    super.onActivityCreated(savedInstanceState);
    if (!isHidden) {
        getData();
        isLoadData = true;
    }
}

@Override
public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
    super.onViewCreated(view, savedInstanceState);
    isViewCreated = true;
}

@Override
public void onHiddenChanged(boolean hidden) {
    super.onHiddenChanged(hidden);
    isHidden = hidden;
    if (!hidden && isViewCreated && !isLoadData) {
        getData();
        isLoadData = true;
    }
}

protected void getData() {}
````



#### Androidx 下的懒加载

虽然之前的方案就能解决轻松的解决 Fragment 的懒加载，但这套方案有一个最大的弊端，就是不可见的 Fragment 也执行了 onResume() 方法。onResume 方法设计的初衷是当执行到该方法时页面就可以与用户进行交互，但是懒加载情况下fragment又是不可见的，这里就与之前的设计有所冲突了。

基于此问题，Google 在 Androidx 在` FragmentTransaction` 中增加了 `setMaxLifecycle `方法来控制 Fragment 所能调用的最大的生命周期函数，该方法可以设置活跃状态下 Fragment 最大的状态，如果该 Fragment 超过了设置的最大状态，那么会强制将 Fragment 降级到正确的状态。

**ViewPager懒加载**

在`FragmentPagerAdapter `与`FragmentStatePagerAdapter`新增了含有 `behavior` 字段的构造函数，如下所示：

```kotlin
public FragmentPagerAdapter(@NonNull FragmentManager fm, @Behavior int behavior) {
    mFragmentManager = fm;
    mBehavior = behavior;
}

public FragmentStatePagerAdapter(@NonNull FragmentManager fm, @Behavior int behavior) {
    mFragmentManager = fm;
    mBehavior = behavior;
}
```

其中 Behavior 的声明如下：

```java
    @Retention(RetentionPolicy.SOURCE)
    @IntDef({BEHAVIOR_SET_USER_VISIBLE_HINT, BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT})
    private @interface Behavior { }

	//当 Fragment 对用户的可见状态发生改变时，setUserVisibleHint 方法会被调用
    @Deprecated
    public static final int BEHAVIOR_SET_USER_VISIBLE_HINT = 0;
	//当前选中的 Fragment 在 Lifecycle.State#RESUMED 状态
    //其他不可见的 Fragment 会被限制在 Lifecycle.State#STARTED 状态
    public static final int BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT = 1;
```

在`FragmentPagerAdapter `的中可以看见实现逻辑

````java
public void setPrimaryItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
    Fragment fragment = (Fragment)object;
    if (fragment != mCurrentPrimaryItem) {
        //如果当前的fragment不是当前选中并可见的Fragment,那么就会调用
    	// setMaxLifecycle 设置其最大生命周期为 Lifecycle.State.STARTED
        if (mCurrentPrimaryItem != null) {
            mCurrentPrimaryItem.setMenuVisibility(false);
            if (mBehavior == BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT) {
                if (mCurTransaction == null) {
                    mCurTransaction = mFragmentManager.beginTransaction();
                }
                mCurTransaction.setMaxLifecycle(mCurrentPrimaryItem, Lifecycle.State.STARTED);
            } else {
                mCurrentPrimaryItem.setUserVisibleHint(false);
            }
        }
        //对于选中的Fragment,则设置其最大生命周期为Lifecycle.State.RESUMED
        fragment.setMenuVisibility(true);
        if (mBehavior == BEHAVIOR_RESUME_ONLY_CURRENT_FRAGMENT) {
            if (mCurTransaction == null) {
                mCurTransaction = mFragmentManager.beginTransaction();
            }
            mCurTransaction.setMaxLifecycle(fragment, Lifecycle.State.RESUMED);
        } else {
            fragment.setUserVisibleHint(true);
        }

        mCurrentPrimaryItem = fragment;
    }
}
````

既然在上述条件下，只有实际可见的 Fragment 会调用 onResume 方法，这样就可以更加简单的实现fragment懒加载

* 只需要在`onResume()`中增加`isLoad`变量默认为false，判断`!isLoad`进行加载数据，加载之后将`isLoad`置为true，就不会再加载

实现代码如下

```kotlin
abstract class LazyFragment : Fragment() {
    private var isLoaded = false

    override fun onResume() {
        super.onResume()
        if (!isLoaded) {
            lazyInit()
            isLoaded = true
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        isLoaded = false
    }

    abstract fun lazyInit()
}
```



**add+hide+show加载Fragment**

这里实现思路其实和`FragmentPagerAdapter `中的`setPrimaryItem`实现是一样

* 将需要显示的 Fragment ，在调用 add 或 show 方法后，`setMaxLifecycle(showFragment, Lifecycle.State.RESUMED)`
* 将需要隐藏的 Fragment ，在调用 hide 方法后，`setMaxLifecycle(fragment, Lifecycle.State.STARTED)`

代码参考如下

```kotlin
/**
 * 使用add+show+hide模式加载fragment
 *
 * 默认显示位置[showPosition]的Fragment，最大Lifecycle为Lifecycle.State.RESUMED
 * 其他隐藏的Fragment，最大Lifecycle为Lifecycle.State.STARTED
 *
 *@param containerViewId 容器id
 *@param showPosition  fragments
 *@param fragmentManager FragmentManager
 *@param fragments  控制显示的Fragments
 */
private fun loadFragmentsTransaction(@IdRes containerViewId: Int, showPosition: Int,
    fragmentManager: FragmentManager, vararg fragments: Fragment) {
    if (fragments.isNotEmpty()) {
        fragmentManager.beginTransaction().apply {
            for (index in fragments.indices) {
                val fragment = fragments[index]
                add(containerViewId, fragment, fragment.javaClass.name)
                if (showPosition == index) {
                    setMaxLifecycle(fragment, Lifecycle.State.RESUMED)
                } else {
                    hide(fragment)
                    setMaxLifecycle(fragment, Lifecycle.State.STARTED)
                }
            }
        }.commit()
    } else {
        throw IllegalStateException("fragments must not empty”)
    }
}

/** 显示需要显示的Fragment[showFragment]，并设置其最大Lifecycle为Lifecycle.State.RESUMED。
 *  同时隐藏其他Fragment,并设置最大Lifecycle为Lifecycle.State.STARTED
 * @param fragmentManager
 * @param showFragment
 */
private fun showHideFragmentTransaction(fragmentManager: FragmentManager, showFragment: Fragment) {
    fragmentManager.beginTransaction().apply {
        show(showFragment)
        setMaxLifecycle(showFragment, Lifecycle.State.RESUMED)

        //获取其中所有的fragment,其他的fragment进行隐藏
        val fragments = fragmentManager.fragments
        for (fragment in fragments) {
            if (fragment != showFragment) {
                hide(fragment)
                setMaxLifecycle(fragment, Lifecycle.State.STARTED)
            }
        }
    }.commit()
}
```





