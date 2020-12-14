####  RecyclerView缓存(二)  ————  RecyclerView缓存管理及应用

从`RecyclerView缓存(一)  —RecyclerView缓存结构`描述的源码中不难发现，5种缓存结构变量均存在Recycler类中，所有ViewHolder缓存的增删改查方法也都在Recycler类中实现。
`A Recycler is responsible for managing scrapped or detached item views for reuse`
正如Recycler类注释描述，Recycler是RecyclerView缓存核心工具类。典型的应用场景就是给LayoutManger提供可复用的视图：
`Typical use of a Recycler by a {@link LayoutManager} will be to obtain views for an adapter's data set representing the data at a given position or item ID`

```java
/**
 * A Recycler is responsible for managing scrapped or detached item views for reuse.
 *
 * <p>A "scrapped" view is a view that is still attached to its parent RecyclerView but
 * that has been marked for removal or reuse.</p>
 *
 * <p>Typical use of a Recycler by a {@link LayoutManager} will be to obtain views for
 * an adapter's data set representing the data at a given position or item ID.
 * If the view to be reused is considered "dirty" the adapter will be asked to rebind it.
 * If not, the view can be quickly reused by the LayoutManager with no further work.
 * Clean views that have not {@link android.view.View#isLayoutRequested() requested layout}
 * may be repositioned by a LayoutManager without remeasurement.</p>
 */
public final class Recycler {
    final ArrayList<ViewHolder> mAttachedScrap = new ArrayList<>();
    ArrayList<ViewHolder> mChangedScrap = null;

    final ArrayList<ViewHolder> mCachedViews = new ArrayList<ViewHolder>();

    private final List<ViewHolder>
            mUnmodifiableAttachedScrap = Collections.unmodifiableList(mAttachedScrap);

    private int mRequestedCacheMax = DEFAULT_CACHE_SIZE;
    int mViewCacheMax = DEFAULT_CACHE_SIZE;

    RecycledViewPool mRecyclerPool;

    private ViewCacheExtension mViewCacheExtension;

    static final int DEFAULT_CACHE_SIZE = 2;
    ...
```

LayoutManager作用是测量和定位RecyclerView中的子视图,也提供策略用于回收不再可见的子视图

关于五种缓存的使用，在tryGetViewHolderForPositionByDeadline方法中，依次从五种缓存数据结构中获取可用缓存：

```java
/**
 * Attempts to get the ViewHolder for the given position, either from the Recycler scrap,
 * cache, the RecycledViewPool, or creating it directly.
 * <p>
 * If a deadlineNs other than {@link #FOREVER_NS} is passed, this method early return
 * rather than constructing or binding a ViewHolder if it doesn't think it has time.
 * If a ViewHolder must be constructed and not enough time remains, null is returned. If a
 * ViewHolder is aquired and must be bound but not enough time remains, an unbound holder is
 * returned. Use {@link ViewHolder#isBound()} on the returned object to check for this.
 *
 * @param position Position of ViewHolder to be returned.
 * @param dryRun True if the ViewHolder should not be removed from scrap/cache/
 * @param deadlineNs Time, relative to getNanoTime(), by which bind/create work should
 *                   complete. If FOREVER_NS is passed, this method will not fail to
 *                   create/bind the holder if needed.
 *
 * @return ViewHolder for requested position
 */
 @Nullable
 ViewHolder tryGetViewHolderForPositionByDeadline(int position,
        boolean dryRun, long deadlineNs) {
    if (position < 0 || position >= mState.getItemCount()) {
        throw new IndexOutOfBoundsException("Invalid item position " + position
                + "(" + position + "). Item count:" + mState.getItemCount()
                + exceptionLabel());
    }
    boolean fromScrapOrHiddenOrCache = false;
    ViewHolder holder = null;
    // 0) If there is a changed scrap, try to find from there
    if (mState.isPreLayout()) {
        holder = getChangedScrapViewForPosition(position);
        fromScrapOrHiddenOrCache = holder != null;
    }
    // 1) Find by position from scrap/hidden list/cache
    if (holder == null) {
        holder = getScrapOrHiddenOrCachedHolderForPosition(position, dryRun);
        if (holder != null) {
            if (!validateViewHolderForOffsetPosition(holder)) {
               // recycle holder (and unscrap if relevant) since it can't be used
                if (!dryRun) {
                    // we would like to recycle this but need to make sure it is not used by
                    // animation logic etc.
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
    if (holder == null) {
        final int offsetPosition = mAdapterHelper.findPositionOffset(position);
        if (offsetPosition < 0 || offsetPosition >= mAdapter.getItemCount()) {
            throw new IndexOutOfBoundsException("Inconsistency detected. Invalid item "
                    + "position " + position + "(offset:" + offsetPosition + ")."
                    + "state:" + mState.getItemCount() + exceptionLabel());
        }

        final int type = mAdapter.getItemViewType(offsetPosition);
        // 2) Find from scrap/cache via stable ids, if exists
        if (mAdapter.hasStableIds()) {
            holder = getScrapOrCachedViewForId(mAdapter.getItemId(offsetPosition),
                    type, dryRun);
            if (holder != null) {
                // update position
                holder.mPosition = offsetPosition;
                fromScrapOrHiddenOrCache = true;
            }
        }
        if (holder == null && mViewCacheExtension != null) {
            // We are NOT sending the offsetPosition because LayoutManager does not
            // know it.
            final View view = mViewCacheExtension
                    .getViewForPositionAndType(this, position, type);
            if (view != null) {
                holder = getChildViewHolder(view);
                if (holder == null) {
                    throw new IllegalArgumentException("getViewForPositionAndType returned"
                            + " a view which does not have a ViewHolder"
                            + exceptionLabel());
                } else if (holder.shouldIgnore()) {
                    throw new IllegalArgumentException("getViewForPositionAndType returned"
                            + " a view that is ignored. You must call stopIgnoring before"
                            + " returning this view." + exceptionLabel());
                }
            }
        }
        if (holder == null) { // fallback to pool
            if (DEBUG) {
                Log.d(TAG, "tryGetViewHolderForPositionByDeadline("
                        + position + ") fetching from shared pool");
            }
            holder = getRecycledViewPool().getRecycledView(type);
            if (holder != null) {
                holder.resetInternal();
                if (FORCE_INVALIDATE_DISPLAY_LIST) {
                    invalidateDisplayListInt(holder);
                }
            }
        }
        if (holder == null) {
            long start = getNanoTime();
            if (deadlineNs != FOREVER_NS
                    && !mRecyclerPool.willCreateInTime(type, start, deadlineNs)) {
                // abort - we have a deadline we can't meet
                return null;
            }
            holder = mAdapter.createViewHolder(RecyclerView.this, type);
            if (ALLOW_THREAD_GAP_WORK) {
                // only bother finding nested RV if prefetching
                RecyclerView innerView = findNestedRecyclerView(holder.itemView);
                if (innerView != null) {
                    holder.mNestedRecyclerView = new WeakReference<>(innerView);
                }
            }

            long end = getNanoTime();
            mRecyclerPool.factorInCreateTime(type, end - start);
            if (DEBUG) {
                Log.d(TAG, "tryGetViewHolderForPositionByDeadline created new ViewHolder");
            }
        }
    }

    // This is very ugly but the only place we can grab this information
    // before the View is rebound and returned to the LayoutManager for post layout ops.
    // We don't need this in pre-layout since the VH is not updated by the LM.
    if (fromScrapOrHiddenOrCache && !mState.isPreLayout() && holder
           .hasAnyOfTheFlags(ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST)) {
        holder.setFlags(0, ViewHolder.FLAG_BOUNCED_FROM_HIDDEN_LIST);
        if (mState.mRunSimpleAnimations) {
           int changeFlags = ItemAnimator
                    .buildAdapterChangeFlagsForAnimations(holder);
            changeFlags |= ItemAnimator.FLAG_APPEARED_IN_PRE_LAYOUT;
            final ItemHolderInfo info = mItemAnimator.recordPreLayoutInformation(mState,
                    holder, changeFlags, holder.getUnmodifiedPayloads());
            recordAnimationInfoIfBouncedHiddenView(holder, info);
        }
    }

    boolean bound = false;
    if (mState.isPreLayout() && holder.isBound()) {
        // do not update unless we absolutely have to.
        holder.mPreLayoutPosition = position;
    } else if (!holder.isBound() || holder.needsUpdate() || holder.isInvalid()) {
        if (DEBUG && holder.isRemoved()) {
            throw new IllegalStateException("Removed holder should be bound and it should"
                    + " come here only in pre-layout. Holder: " + holder
                    + exceptionLabel());
        }
        final int offsetPosition = mAdapterHelper.findPositionOffset(position);
        bound = tryBindViewHolderByDeadline(holder, offsetPosition, position, deadlineNs);
   }

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
}

```

**RecyclerView缓存作用**

1、mChangedScrap和mAttachedScrap

> mChangedScrap和mAttachedScrap集合列表在RecyclerView内部使用,不对外暴露（即使用层无可用方法控制），主要在RecyclerView内部布局（onLayout)子视图时会使用到，用作临时存储。mChangedScrap和mAttachedScrap在这不作深入分析。
>

2、mCachedViews

> mCachedViews中缓存的ViewHolder在使用时无需调用onBindViewHolder方法进行视图数据绑定，可完全复用，但mCachedViews中获取的ViewHolder也只能用于固定position位置的复用（mCachedViews中的ViewHolder都会有固定绑定好的position）。默认最大缓存个数mViewCacheMax = DEFAULT_CACHE_SIZE =2，但实际在RecyclerView列表数据填充之后进行上下（或左右）滑动时，mCachedViews数量会有3个，原因是RecyclerView的prefech机制会导致在mCachedViews中会额外增加一个ViewHolder的缓存。
>

3、mViewCacheExtension

> 上文的缓存结构分析可知，ViewCacheExtension是暴露给应用层实现的自定义缓存，使用场景是某一类相同viewType不同位置的子View，要保证在滑动中始终存在于内存中并且不会出现重新绑定视图数据（即重复调用onBindViewHolder）的情况。无法使用mCachedViews的原因是，尽管mCachedViews也不需要重新绑定视图数据，但mCachedViews的缓存复用和移除不固定viewType类型，并且和position强绑定，mCachedViews缓存的是最近滑出屏幕的子视图。
>

4、mRecyclerPool

> mRecyclerPool与mCachedViews不同的是内部缓存的ViewHolder在使用时需要调用onBindViewHolder方法重新进行视图数据绑定，mRecyclerPool中缓存的所有ViewHolder都是被清除状态无绑定postisin。因为重新调用onBindViewHolder方法进行视图数据绑定，所以使用mRecyclerPool中的ViewHolder缓存是必然会重新绑定视图数据，再次调用onBindViewHolder方法。mRecyclerPool缓存主要左右是减少ViewHolder创建即减少onCreateViewHolder方法的调用
>

**Recycler缓存使用方式**

1、mCachedViews

> mCachedViews的使用相对简单，使用层直接控制的方法只有setItemViewCacheSize即设置mCachedViews的最大缓存个数,
> `mRecyclerView. setItemViewCacheSize(maxCacheSize);`

```java
/**
 * Set the number of offscreen views to retain before adding them to the potentially shared
 * {@link #getRecycledViewPool() recycled view pool}.
 *
 * <p>The offscreen view cache stays aware of changes in the attached adapter, allowing
 * a LayoutManager to reuse those views unmodified without needing to return to the adapter
 * to rebind them.</p>
 *
 * @param size Number of views to cache offscreen before returning them to the general
 *             recycled view pool
 */
public void setItemViewCacheSize(int size) {
    mRecycler.setViewCacheSize(size);
}
```

2、ViewCacheExtension

> ViewCacheExtension自定义缓存使用核心是理解作用和使用时机，使用demo：

```java
SparseArray<View> specials = new SparseArray<>();
...

recyclerView.getRecycledViewPool().setMaxRecycledViews(SPECIAL, 0);

recyclerView.setViewCacheExtension(new RecyclerView.ViewCacheExtension() {
   @Override
   public View getViewForPositionAndType(RecyclerView.Recycler recycler,
                                         int position, int type) {
       return type == SPECIAL ? specials.get(position) : null;
   }
});

...
class SpecialViewHolder extends RecyclerView.ViewHolder {
	   ...		
   public void bindTo(int position) {
       ...
       specials.put(position, itemView);
   }
}
```

3、RecycledViewPool

> RecyclerdViewPool具体使用方式：

```java
RecyclerView.RecycledViewPool recycledViewPool = new RecyclerView.RecycledViewPool();

//或 RecyclerView.RecycledViewPool recycledViewPool = mRecyclerView.getRecycledViewPool();

recycledViewPool.setMaxRecycledViews(type1, cacheSize1);

recycledViewPool.setMaxRecycledViews(type2, cacheSize2);

recycledViewPool.setMaxRecycledViews(type3, cacheSize3);

...

mRecyclerView.setRecycledViewPool(recycledViewPool);

```

**结语**

对RecyclerView缓存体系的梳理，会对日常项目开发列表相关业务有更深入的理解。这里仅做了相对简单的说明，要系统深入理解RecyclerView缓存，建议用最简单的RecyclerView列表展示demo，在RecyclerView适配器的核心方法如onCreateViewHolder、onBindViewHolder等加log，观察创建、滑动中的具体日志输出规律。并结合RecyclerView缓存的5大变量进行debug才会对RecyclerView的整个缓存体系有更加深入的理解。