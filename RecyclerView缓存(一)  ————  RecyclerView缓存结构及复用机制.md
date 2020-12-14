####  RecyclerView缓存(一)  ————  RecyclerView缓存结构及复用机制

**一、概要**

RecyclerView是Android开发中一个至关重要的UI控件，在日常项目的业务开发中无处不在，功能也极其强大。子View不同逻辑解耦，view回收复用高性能，易用性体现在局部刷新、item动画，拖拽测滑等，基本能替代ListView所有功能(但也并不能完全替代ListView，ListView并没有被标记为@Deprecated。RecyclerView核心优势是缓存机制的设计，本文以RecyclerView缓存原理为主线，部分源码进行分析，从RecyclerView的缓存结构，缓存管理以及缓存使用等方面进行展开。

> RecylerView缓存的简单梳理，RecylerView中一共有五种缓存，分别是：
>
> - mScrapView
> - mAttachedScrap
> - mCachedViews
> - mViewCacheExtension
> - mRecyclerPool
>

其中前两种mScrapView、mAttachedScrap并不对外暴露，真正开发中能控制或自定义的是后三种mCachedViews、mViewCacheExtension和mRecyclerPool，所以在学习RecyclerView缓存原理的过程中，建议的方向是：理解前两种的作用以及相关源码，理解后三者的作用、源码并掌握实际用法。在阅读理解过程中结合实践对关键方法和变量进行跟踪debug，会更快的掌握整个知识体系。

> RecyclerView缓存本质上指的ViewHolder缓存，下面是源码中五种缓存变量的数据结构：

- **mChangedScrap**

>  这个没理解是干嘛用的，看名字应该跟 ViewHolder 的数据发生变化时有关吧，在 RecyclerView 滑动的过程中，也没有发现到这里找复用的 ViewHolder，所以这个可以先暂时放一边
>

```java
ArrayList<ViewHolder> mChangedScrap = null;
```

- **mAttachedScrap**

> 用于缓存显示在屏幕上的 item 的 ViewHolder，场景好像是 RecyclerView 在 onLayout 时会先把 children 都移除掉，再重新添加进去，所以这个 List 应该是用在布局过程中临时存放 children 的，反正在 RecyclerView 滑动过程中不会在这里面来找复用的 ViewHolder 就是了
>

```java
final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
```

- **mCachedViews**

> 这个就重要得多了，滑动过程中的回收和复用都是先处理的这个 List，这个集合里存的 ViewHolder 的原本数据信息都在，所以可以直接添加到 RecyclerView 中显示，不需要再次重新 onBindViewHolder()。
>

```java
final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();
```

- **mViewCacheExtension**

> ViewCacheExtension是一个abstranct类，暴露给应用层实现，只有一个abstract的getViewForPositionAndType方法需要覆写。

```java
private ViewCacheExtension mViewCacheExtension;
/**
   * ViewCacheExtension is a helper class to provide an additional layer of view caching that can
   * be controlled by the developer.
   * <p>
   * When {@link Recycler#getViewForPosition(int)} is called, Recycler checks attached scrap and
   * first level cache to find a matching View. If it cannot find a suitable View, Recycler will
   * call the {@link #getViewForPositionAndType(Recycler, int, int)} before checking
   * {@link RecycledViewPool}.
   * <p>
   * Note that, Recycler never sends Views to this method to be cached. It is developers
   * responsibility to decide whether they want to keep their Views in this custom cache or let
   * the default recycling policy handle it.
   */
public abstract static class ViewCacheExtension {
      /**
       * Returns a View that can be binded to the given Adapter position.
       * <p>
       * This method should <b>not</b> create a new View. Instead, it is expected to return
       * an already created View that can be re-used for the given type and position.
       * If the View is marked as ignored, it should first call
       * {@link LayoutManager#stopIgnoringView(View)} before returning the View.
       * <p>
       * RecyclerView will re-bind the returned View to the position if necessary.
       *
       * @param recycler The Recycler that can be used to bind the View
       * @param position The adapter position
       * @param type     The type of the View, defined by adapter
       * @return A View that is bound to the given position or NULL if there is no View to re-use
       * @see LayoutManager#ignoreView(View)
       */
      @Nullable
      public abstract View getViewForPositionAndType(@NonNull Recycler recycler, int position,int type);
  }
```

- **mRecyclerPool**

> 这个也很重要，但存在这里的 ViewHolder 的数据信息会被重置掉，相当于 ViewHolder 是一个重创新建的一样，所以需要重新调用 onBindViewHolder 来绑定数据。
>

```java
/**
  * RecycledViewPool lets you share Views between multiple RecyclerViews.
  * <p>
  * If you want to recycle views across RecyclerViews, create an instance of RecycledViewPool
  * and use {@link RecyclerView#setRecycledViewPool(RecycledViewPool)}.
  * <p>
  * RecyclerView automatically creates a pool for itself if you don't provide one.
  */
 public static class RecycledViewPool {
     private static final int DEFAULT_MAX_SCRAP = 5;
     /**
      * Tracks both pooled holders, as well as create/bind timing metadata for the given type.
      *
      * Note that this tracks running averages of create/bind time across all RecyclerViews
      * (and, indirectly, Adapters) that use this pool.
      *
      * 1) This enables us to track average create and bind times across multiple adapters. Even
      * though create (and especially bind) may behave differently for different Adapter
      * subclasses, sharing the pool is a strong signal that they'll perform similarly, per type.
      *
      * 2) If {@link #willBindInTime(int, long, long)} returns false for one view, it will return
      * false for all other views of its type for the same deadline. This prevents items
      * constructed by {@link GapWorker} prefetch from being bound to a lower priority prefetch.
      */
     static class ScrapData {
         final ArrayList<ViewHolder> mScrapHeap = new ArrayList<>();
         int mMaxScrap = DEFAULT_MAX_SCRAP;
         long mCreateRunningAverageNs = 0;
         long mBindRunningAverageNs = 0;
     }
     SparseArray<ScrapData> mScrap = new SparseArray<>();
        
    ...
       
     /**
      * Sets the maximum number of ViewHolders to hold in the pool before discarding.
      *
      * @param viewType ViewHolder Type
      * @param max Maximum number
      */
     public void setMaxRecycledViews(int viewType, int max) {
         ScrapData scrapData = getScrapDataForType(viewType);
         scrapData.mMaxScrap = max;
         final ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
         while (scrapHeap.size() > max) {
             scrapHeap.remove(scrapHeap.size() - 1);
         }
     }

     /**
      * Returns the current number of Views held by the RecycledViewPool of the given view type.
      */
     public int getRecycledViewCount(int viewType) {
         return getScrapDataForType(viewType).mScrapHeap.size();
     }

     /**
      * Acquire a ViewHolder of the specified type from the pool, or {@code null} if none are
      * present.
      *
      * @param viewType ViewHolder type.
      * @return ViewHolder of the specified type acquired from the pool, or {@code null} if none
      * are present.
      */
     @Nullable
     public ViewHolder getRecycledView(int viewType) {
         final ScrapData scrapData = mScrap.get(viewType);
         if (scrapData != null && !scrapData.mScrapHeap.isEmpty()) {
             final ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
             return scrapHeap.remove(scrapHeap.size() - 1);
         }
         return null;
     }

    ...

    private ScrapData getScrapDataForType(int viewType) {
         ScrapData scrapData = mScrap.get(viewType);
         if (scrapData == null) {
             scrapData = new ScrapData();
             mScrap.put(viewType, scrapData);
         }
         return scrapData;
     }
 }
```

简单分析RecyclerPool类的结构，里面核心数据结构：

```java
SparseArray<ScrapData> mScrap = new SparseArray<>();
```

> SparseArray仅用于存储key（int），value（object）的结构，比HashMap更省内存，在某些条件下性能更好（具体本文不展开）。再看ScrapData结构为：
>

```java
static class ScrapData {
    final ArrayList<ViewHolder> mScrapHeap = new ArrayList<>();
    int mMaxScrap = DEFAULT_MAX_SCRAP;
    long mCreateRunningAverageNs = 0;
    long mBindRunningAverageNs = 0;
}
```

ScrapData类需要注意两个维护的变量：

> ```java
> final ArrayList<ViewHolder> mScrapHeap = new ArrayList<>();
> 
> int mMaxScrap = DEFAULT_MAX_SCRAP;
> ```

看到了熟悉的ViewHolder集合列表，以及列表缓存默认最大个数`DEFAULT_MAX_SCRAP = 5`，再看看ScrapData对象被初始化创建时机：

```java
private ScrapData getScrapDataForType(int viewType) {
    ScrapData scrapData = mScrap.get(viewType);
    if (scrapData == null) {
        scrapData = new ScrapData();
        mScrap.put(viewType, scrapData);
    }
    return scrapData;
}
```

显然是在getScrapDataForType方法传入viewType时创建，再找找getScrapDataForType被调用的地方会发现有一个对外暴露的方法setMaxRecycledViews里面调用,从而说明可以根据viewType设置不同类别ViewHolder的最大缓存个数：

```java
public void setMaxRecycledViews(int viewType, int max) {
    ScrapData scrapData = getScrapDataForType(viewType);
    scrapData.mMaxScrap = max;
    final ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
    while (scrapHeap.size() > max) {
        scrapHeap.remove(scrapHeap.size() - 1);
    }
}
```

RecycledViewPool里还有一个需要注意的方法getRecycledView：

```java
@Nullable
public ViewHolder getRecycledView(int viewType) {
     final ScrapData scrapData = mScrap.get(viewType);
     if (scrapData != null && !scrapData.mScrapHeap.isEmpty()) {
        final ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
        return scrapHeap.remove(scrapHeap.size() - 1);
     }
     return null;
}
```

这个方法是RecyclerView使用时比较熟悉的方法，根据viewType获取ViewHolder，可以看出本质是从RecycledViewPool的mScrap缓存结构中获取ViewHolder缓存。至此，整个RecyclerView的缓存结构大致梳理清楚。

**复用机制**

getViewForPosition()

> 这个方法是复用机制的入口，也就是 Recycler 开放给外部使用复用机制的api，外部调用这个方法就可以返回想要的 View，而至于这个 View 是复用而来的，还是重新创建得来的，就都由 Recycler 内部实现，对外隐藏。

```java
//入口在这里 
public View getViewForPosition(int position) { 
    return getViewForPosition(position, false); 
} 

View getViewForPosition(int position, boolean dryRun) { 
    return tryGetViewHolderForPositionByDeadline(position, dryRun, FOREVER_NS).itemView; 
} 

ViewHolder tryGetViewHolderForPositionByDeadline(int position, 
            boolean dryRun, long deadlineNs) {  
    //复用机制工作原理都在这里 
    //... 
}
```

tryGetViewHolderForPositionByDeadline()

> 所以，Recycler 的复用机制内部实现就在这个方法里。分析逻辑之前，先看一下 Recycler 的几个结构体，用来缓存 ViewHolder 的。

```java
 public final class Recycler { 
    final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>(); 
    ArrayList<ViewHolder> mChangedScrap = null; 
    final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>(); 

    private final List<ViewHolder> 
            mUnmodifiableAttachedScrap = Collections.unmodifiableList(mAttachedScrap); 

    private int mRequestedCacheMax = DEFAULT_CACHE_SIZE; 
    int mViewCacheMax = DEFAULT_CACHE_SIZE; 
    //这个是重点，在上文中有介绍 
    RecycledViewPool mRecyclerPool; 

    private ViewCacheExtension mViewCacheExtension; 

    static final int DEFAULT_CACHE_SIZE = 2; 
 }
```

复用的逻辑：

```java
ViewHolder tryGetViewHolderForPositionByDeadline(int position, 
                boolean dryRun, long deadlineNs) { 
    if (position < 0 || position >= mState.getItemCount()) { 
        throw new IndexOutOfBoundsException("Invalid item position " + position 
                + "(" + position + "). Item count:" + mState.getItemCount()); 
    } 
    //...省略代码 
}
```

第一步很简单，position 如果在 item 的范围之外的话，那就抛异常吧。继续往下看:

```java
ViewHolder tryGetViewHolderForPositionByDeadline(int position, 
                boolean dryRun, long deadlineNs) { 
    //...省略看过的代码 
    boolean fromScrapOrHiddenOrCache = false; 
    ViewHolder holder = null; 
    // 0) If there is a changed scrap, try to find from there 
    //上面是Google留的注释(emmm，这里我也没理解) 
    if (mState.isPreLayout()) { 
        holder = getChangedScrapViewForPosition(position); 
        fromScrapOrHiddenOrCache = holder != null; 
    } 
}
```

如果是在 isPreLayout() 时，那么就去 mChangedScrap 中找。

```java
ViewHolder tryGetViewHolderForPositionByDeadline(int position, 
                boolean dryRun, long deadlineNs) { 
    //...省略看过的代码 
    // 1) Find by position from scrap/hidden list/cache 
    if (holder == null) { 
        //这里是第一次找可复用的ViewHolder了，得跟进去看看 
        holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun); 
        //... 
    } 
}

ViewHolder getScrapOrHiddenOrCachedHolderForPosition(int position,boolean dryRun) { 
    final int scrapCount = mAttachedScrap.size(); 

    // Try first for an exact, non-invalid match from scrap. 
    for (int i = 0; i < scrapCount; i++) { 
        //首先去mAttachedScrap中遍历寻找，匹配条件也很多 
        final ViewHolder holder = mAttachedScrap.get(i); 
        if (!holder.wasReturnedFromScrap() && holder.getLayoutPosition() == position 
                && !holder.isInvalid() && (mState.mInPreLayout || !holder.isRemoved())) { 
            holder.addFlags(ViewHolder.FLAG_RETURNED_FROM_SCRAP); 
            return holder; 
        } 
    } 
    //省略代码... 
}
```

> 首先，去 mAttachedScrap 中寻找 position 一致的 viewHolder，需要匹配一些条件，大致是这个 viewHolder 没有被移除，是有效的之类的条件，满足就返回这个 viewHolder。所以，这里的关键就是要理解这个 mAttachedScrap 到底是什么，存的是哪些 ViewHolder。一次遥控器按键的操作，不管有没有发生滑动，都会导致 RecyclerView 的重新 onLayout，那要 layout 的话，RecyclerView 会先把所有 children 先 remove 掉，然后再重新 add 上去，完成一次 layout 的过程。那么这暂时性的 remove 掉的 viewHolder 要存放在哪呢，就是放在这个 mAttachedScrap 中了，这就是我的理解了。所以，感觉这个 mAttachedScrap 中存放的 viewHolder 跟回收和复用关系不大。
>
> 网上一些分析的文章有说，RecyclerView 在复用时会按顺序去 mChangedScrap, mAttachedScrap 等等缓存里找，没有找到再往下去找，从代码上来看是这样没错，但我觉得这样表述有问题。因为就我们这篇文章基于 RecyclerView 的滑动场景来说，新卡位的复用以及旧卡位的回收机制，其实都不会涉及到 mChangedScrap 和 mAttachedScrap，所以我觉得还是基于某种场景来分析相对应的回收复用机制会比较好。就像 mChangedScrap 我虽然没理解是干嘛用的，但我猜测应该是在当数据发生变化时才会涉及到的复用场景，所以当我分析基于滑动场景时的复用时，即使我对这块不理解，影响也不会很大。

```java
ViewHolder getScrapOrHiddenOrCachedHolderForPosition(int position,boolean dryRun) { 
    //...省略看过的代码 
    // Search in our first-level recycled view cache. 
    //下面就是重点了，去mCachedViews里遍历 
    final int cacheSize = mCachedViews.size(); 
    for (int i = 0; i < cacheSize; i++) { 
        final ViewHolder holder = mCachedViews.get(i); 
        // invalid view holders may be in cache if adapter has stable ids as they can be 
        // retrieved via getScrapOrCachedViewForId 
        // 上面的大意是即使是失效的holser也有可能可以拿来复用，但需要我们重写adapter的setHasStadleId并且提供一个id时，在getScrapOrCachedViewForId()里就可以再去mCachedViews里找一遍。   
        if (!holder.isInvalid() && holder.getLayoutPosition() == position) { 
            if (!dryRun) { //dryRun一直为false 
                mCachedViews.remove(i);//所以，如果position匹配，那么就将这个ViewHolder移除mCachedViews 
            } 
            if (DEBUG) { 
                Log.d(TAG, "getScrapOrHiddenOrCachedHolderForPosition(" + position 
                        + ") found match in cache: " + holder); 
            } 
            return holder; 

    } 
    return null; 
}
```

> 滑动场景中的复用会用到这里的机制。mCachedViews 的大小默认为2。遍历 mCachedViews，找到 position 一致的 ViewHolder，mCachedViews 里存放的 ViewHolder 的数据信息都保存着，所以 mCachedViews 可以理解成，只有原来的卡位可以重新复用这个 ViewHolder，新位置的卡位无法从 mCachedViews 里拿 ViewHolder出来用。

```java

ViewHolder tryGetViewHolderForPositionByDeadline(int position, 
                boolean dryRun, long deadlineNs) { 
    //...省略看过的代码 
    // 1) Find by position from scrap/hidden list/cache 
    if (holder == null) { 
        //这里是第一次找可复用的ViewHolder了，得跟进去看看 
        holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun); 
        //之前分析跟进了上面那个方法，找到ViewHolder后 
        if (holder != null) { 
            //需要再次验证一下这个ViewHodler是否可以拿来复用 
            if (!validateViewHolderForOffsetPosition(holder)) { 
                // recycle holder (and unscrap if relevant) since it can't be used 
                if (!dryRun) { 
                    // we would like to recycle this but need to make sure it is not used by 
                    // animation logic etc. 
                    //如果不能复用，就把它要么仍到mAttachedScrap或者扔到ViewPool里 
                    holder.addFlags(ViewHolder.FLAG_INVALID); 
                    if (holder.isScrap()) { 
                        removeDetachedView(holder.itemView, false); 
                        holder.unScrap(); 
                    } else if (holder.wasReturnedFromScrap()) { 
                        holder.clearReturnedFromScrapFlag(); 
                    } 
                    recycleViewHolderInternal(holder); 
                } 
                holder = null; 
            } else { 
                fromScrapOrHiddenOrCache = true; 
            } 
        } 
    } 
}

//就算 position 匹配找到了 ViewHolder，还需要判断一下这个 ViewHolder 是否已经被 remove 掉，type 类型一致不一致
boolean validateViewHolderForOffsetPosition(ViewHolder holder) { 
    // if it is a removed holder, nothing to verify since we cannot ask adapter anymore 
    // if it is not removed, verify the type and id. 
    if (holder.isRemoved()) { 
        if (DEBUG && !mState.isPreLayout()) { 
            throw new IllegalStateException("should not receive a removed view unless it" 
                    + " is pre layout"); 
        } 
        return mState.isPreLayout(); 
    } 
    if (holder.mPosition < 0 || holder.mPosition >= mAdapter.getItemCount()) { 
        throw new IndexOutOfBoundsException("Inconsistency detected. Invalid view holder " 
                + "adapter position" + holder); 
    } 
    //如果type类型不一样，那就不能复用 
    if (!mState.isPreLayout()) { 
        // don't check type if it is pre-layout. 
        final int type = mAdapter.getItemViewType(holder.mPosition); 
        if (type != holder.getItemViewType()) { 
            return false; 
        } 
    } 
    if (mAdapter.hasStableIds()) { 
        return holder.getItemId() == mAdapter.getItemId(holder.mPosition); 
    } 
    return true;     
}


//在 mCachedViews 中寻找，没有找到的话，就继续再找一遍，刚才是通过 position 来找，那这次就换成id，然后重复上面的步骤再找一遍
ViewHolder tryGetViewHolderForPositionByDeadline(int position, 
                boolean dryRun, long deadlineNs) { 
    //...省略看过的代码 
    if (holder == null) { 
        final int offsetPosition = mAdapterHelper.findPositionOffset(position); 
        if (offsetPosition < 0 || offsetPosition >= mAdapter.getItemCount()) { 
            throw new IndexOutOfBoundsException("//省略..."); 
        } 

        final int type = mAdapter.getItemViewType(offsetPosition); 
        // 2) Find from scrap/cache via stable ids, if exists 
        if (mAdapter.hasStableIds()) {//如果有设置stableIs，就再从Scrap和cached里根据id找一次 
            holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition), 
                   type, dryRun); 
            if (holder != null) { 
                // update position 
                holder.mPosition = offsetPosition; 
                fromScrapOrHiddenOrCache = true; 
            } 
        } 
        //省略之后步骤，后续分析... 
    } 
}
```

> getScrapOrCachedViewForId() 做的事跟 getScrapOrHiddenOrCacheHolderForPosition() 其实差不多，只不过一个是通过 position 来找 ViewHolder，一个是通过 id 来找。而这个 id 并不是我们在 xml 中设置的 android:id， 而是 Adapter 持有的一个属性，默认是不会使用这个属性的，所以这里其实是不会执行的，除非我们重写了 Adapter 的 setHasStableIds()

```java
ViewHolder tryGetViewHolderForPositionByDeadline(int position, 
                boolean dryRun, long deadlineNs) { 
    //...省略看过的代码 
    if (holder == null) { 
        final int offsetPosition = mAdapterHelper.findPositionOffset(position); 
        //省略无关代码... 
        final int type = mAdapter.getItemViewType(offsetPosition); 
        //省略看过的的代码... 
        //这里开始就又去另一个地方找了，RecycledViewPool 
        if (holder == null) { // fallback to pool 
            if (DEBUG) { 
                Log.d(TAG, "tryGetViewHolderForPositionByDeadline("+ position + ") fetching from shared pool"); 
            } 
            //跟进这个方法看看 
            holder = getRecycledViewPool().getRecycledView(type); 
            if (holder != null) { 
                //如果在ViewPool里找到可复用的ViewHolder，那就重置ViewHolder的数据，这样ViewHolder就可以当作全新的来使用了 
                holder.resetInternal(); 
                if (FORCE_INVALIDATE_DISPLAY_LIST) { 
                    invalidateDisplayListInt(holder); 
                } 
            } 
        } 
        //省略之后步骤，后续分析... 
    } 
}
//这里是去 RecyclerViewPool 里取 ViewHolder，ViewPool 会根据不同的 item type 创建不同的 List，每个 List 默认大小为5个
```

> ViewPool 会根据不同的 viewType 创建不同的集合来存放 ViewHolder，那么复用的时候，只要 ViewPool 里相同的 type 有 ViewHolder 缓存的话，就将最后一个拿出来复用，不用像 mCachedViews 需要各种匹配条件，只要有就可以复用。拿到 ViewHolder 之后，还会再次调用 resetInternal() 来重置 ViewHolder，这样 ViewHolder 就可以当作一个全新的 ViewHolder 来使用了，这也就是为什么从这里拿的 ViewHolder 都需要重新 onBindViewHolder() 了。
>
> 如果 ViewPool 中都没有找到 ViewHolder 来使用的话，那就调用 Adapter 的 onCreateViewHolder 来创建一个新的 ViewHolder 使用。上面一共有很多步骤来找 ViewHolder，不管在哪个步骤，只要找到 ViewHolder 的话，那下面那些步骤就不用管了，然后都要继续往下判断是否需要重新绑定数据，还有检查布局参数是否合法。

``` java
ViewHolder tryGetViewHolderForPositionByDeadline(int position,  
                boolean dryRun, long deadlineNs) {  
    //...省略上述分析的找ViewHolder的代码...  
    //代码执行到这里，ViewHolder肯定不为Null了，因为就算在各个缓存里没找到，最后一步也会重新创建一个  
    boolean bound = false;  
    if (mState.isPreLayout() && holder.isBound()) {  
        // do not update unless we absolutely have to.  
        holder.mPreLayoutPosition = position;  
    } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {  
        if (DEBUG && holder.isRemoved()) {  
            throw new IllegalStateException("Removed holder should be bound and it should" + " come here only in pre-layout. Holder: " + holder);  
        }  
        final int offsetPosition = mAdapterHelper.findPositionOffset(position);  
        //调用Adapter.onBindViewHolder()来重新绑定数据  
        bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs);  
    }  
    //下面是验证itemView的布局参数是否可用，并设置可用的布局参数  
    final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();  
    final LayoutParams rvLayoutParams;  
    if (lp == null) {  
        rvLayoutParams = (LayoutParams) generateDefaultLayoutParams();  
        holder.itemView.setLayoutParams(rvLayoutParams);  
    } else if (!checkLayoutParams(lp)) {  
        rvLayoutParams = (LayoutParams) generateLayoutParams(lp);  
        holder.itemView.setLayoutParams(rvLayoutParams);  
    } else {  
        rvLayoutParams = (LayoutParams) lp;  
    }  
    rvLayoutParams.mViewHolder = holder;  
    rvLayoutParams.mPendingInvalidate = fromScrapOrHiddenOrCache && bound;  
    return holder;  
    //结束  
}
```

> 到tryGetViewHolderForPositionByDeadline() 这个方法就结束了。这大概就是 RecyclerView 的复用机制，中间跳过很多地方，因为 RecyclerView 有各种场景可以刷新他的 view，比如重新 setLayoutManager()，重新 setAdapter()，或者 notifyDataSetChanged()，或者滑动等等之类的场景，只要重新layout，就会去回收和复用 ViewHolder，所以这个复用机制需要考虑到各种各样的场景。把代码一行行的啃透有点吃力，所以我就只借助 RecyclerView 的滑动的这种场景来分析它涉及到的回收和复用机制。关于回收机制在下篇介绍。
>

**总结**

RecyclerView 滑动场景下的回收复用涉及到的结构体两个：mCachedViews 和 RecyclerViewPool。mCachedViews 优先级高于 RecyclerViewPool，回收时，最新的 ViewHolder 都是往 mCachedViews 里放，如果它满了，那就移出一个扔到 ViewPool 里好空出位置来缓存最新的 ViewHolder。复用时，也是先到 mCachedViews 里找 ViewHolder，但需要各种匹配条件，概括一下就是只有原来位置的卡位可以复用存在 mCachedViews 里的 ViewHolder，如果 mCachedViews 里没有，那么才去 ViewPool 里找。在 ViewPool 里的 ViewHolder 都是跟全新的 ViewHolder 一样，只要 type 一样，有找到，就可以拿出来复用，重新绑定下数据即可

