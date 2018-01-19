# **SwipeRecyclerView学习之路（四）**
#### *本着初学者的精神 哪怕再简单的东西都要记下*

概述
--------------------------------
本篇将介绍应用最为广泛的下拉刷新和加载更多在SwipeRecyclerView中的运用与分析

运用
--------------------------------
#### 加载更多 ####
首先需要调用useDefaultLoadMore方法使用默认的加载更多的View，当然也可以自定义加载更多的View（这个后面再说），其次需要设置加载更多的监听，最后还需要取消
掉自动加载更多，至于为什么取消掉自动加载更多，后面再分析。
>

        /** 加载更多 **/
        swipeMenu_rv.useDefaultLoadMore(); //使用默认的加载更多的View
        swipeMenu_rv.setLoadMoreListener(mLoadMoreListener); //设置加载更多的监听
        swipeMenu_rv.setAutoLoadMore(false);//取消自动加载更多

那么问题来了，监听回调里到底是什么呢
>

    private SwipeMenuRecyclerView.LoadMoreListener mLoadMoreListener = new SwipeMenuRecyclerView.LoadMoreListener() {
        @Override
        public void onLoadMore() {
            //加载更多
            int num = books.size();
            if (num<100){
                for (int i = num;i<10+num;i++){
                    Book book = new Book();
                    book.setName("recycler item " + (i + 1));
                    books.add(book);
                }
                recyclerAdapter.notifyDataSetChanged();
                // 加载完更多数据，一定要调用这个方法。
                // 第一个参数：表示此次数据是否为空。
                // 第二个参数：表示是否还有更多数据。
                swipeMenu_rv.loadMoreFinish(false, true);
            }else {
                recyclerAdapter.notifyDataSetChanged();
                // 加载完更多数据，一定要调用这个方法。
                // 第一个参数：表示此次数据是否为空。
                // 第二个参数：表示是否还有更多数据。
                swipeMenu_rv.loadMoreFinish(true, false);
            }

        }
    };

从代码可以看出该监听是SwipeMenuRecyclerView中的LoadMoreListener，并重写了onLoadMore,在其中我们需要加载数据并通知SwipeRecyclerView列表更新，最后
则必须调用loadMoreFinish方法，其有两个boolean参数，第一个参数表示此次数据是否为空，第二个参数表示是否还有更多的数据。至于为什么调用该方法，又起到了
什么作用，我们待会儿一一分析，至此运行下代码，加载更多功能就可以正常工作了。

自定义加载更多View也很简单，只需要自定义一个View，并实现一个接口即可：
>

    public class DefineLoadMoreView extends LinearLayout implements SwipeMenuRecyclerView.LoadMoreView,View.OnClickListener {

        private LoadMoreListener mLoadMoreListener;

        public DefineLoadMoreView(Context context) {
            super(context);
            ...
            setOnClickListener(this);
       }

        /**
         * 马上开始回调加载更多了，这里应该显示进度条。
         */
        @Override
        public void onLoading() {
            // 展示加载更多的动画和提示信息。
            ...
        }
    
        /**
         * 加载更多完成了。
         *
         * @param dataEmpty 是否请求到空数据。
         * @param hasMore   是否还有更多数据等待请求。
         */
        @Override
        public void onLoadFinish(boolean dataEmpty, boolean hasMore) {
            // 根据参数，显示没有数据的提示、没有更多数据的提示。
            // 如果都不存在，则都不用显示。
        }
    
        /**
         * 加载出错啦，下面的错误码和错误信息二选一。
         *
         * @param errorCode    错误码。
         * @param errorMessage 错误信息。
         */
        @Override
        public void onLoadError(int errorCode, String errorMessage) {
        }
    
        /**
         * 调用了setAutoLoadMore(false)后，在需要加载更多的时候，此方法被调用，并传入listener。
         */
        @Override
        public void onWaitToLoadMore(SwipeMenuRecyclerView.LoadMoreListener loadMoreListener) {
            this.mLoadMoreListener = loadMoreListener;
            }
    
        /**
         * 非自动加载更多时mLoadMoreListener才不为空。
         */
        @Override
        public void onClick(View v) {
            if (mLoadMoreListener != null) mLoadMoreListener.onLoadMore();
        }
    }

接下来就是初始化工作了
>

        // 自定义的核心就是DefineLoadMoreView类。
        DefineLoadMoreView loadMoreView = new DefineLoadMoreView(this);
        mRecyclerView.addFooterView(loadMoreView); // 添加为Footer。
        mRecyclerView.setLoadMoreView(loadMoreView); // 设置LoadMoreView更新监听。
        mRecyclerView.setLoadMoreListener(mLoadMoreListener); // 加载更多的监听。


#### 下拉刷新 ####
至于下拉刷新就更简单了，在android.support.v4包中有一个SwipeRefreshLayout,当你需要用到下拉刷新的时候，直接将其嵌套在需要刷新的布局之外。
>

    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/refresh_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <com.yanzhenjie.recyclerview.swipe.SwipeMenuRecyclerView
            android:id="@+id/recycler_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
    </android.support.v4.widget.SwipeRefreshLayout>

初始化布局并调用setOnRefreshListener方法设置监听，并在监听回调方法onRefresh中做刷新相关的回调工作
>

        mRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.refresh_layout);
        mRefreshLayout.setOnRefreshListener(mRefreshListener); // 刷新监听。
        
        
    /**
     * 刷新。
     */
    private SwipeRefreshLayout.OnRefreshListener mRefreshListener = new SwipeRefreshLayout.OnRefreshListener() {
        @Override
        public void onRefresh() {
            mRecyclerView.postDelayed(new Runnable() {
                @Override
                public void run() {
                    loadData();
                }
            }, 1000); // 延时模拟请求服务器。
        }
    };
   
分析
--------------------------------
#### 加载更多 ####
首先来看看第一步useDefaultLoadMore代码中是个什么
>

    public void useDefaultLoadMore() {
        DefaultLoadMoreView defaultLoadMoreView = new DefaultLoadMoreView(getContext());
        addFooterView(defaultLoadMoreView);
        setLoadMoreView(defaultLoadMoreView);
    }

是不是似曾相识，对的在前面我们说自定义加载更多的时候也是这三行代码，只不够DefaultLoadMoreView换成了自定义的LoadMoewView而已。addFooterView方法的
实现在上篇中已经说过，这里的目的是将LoadMoreView添加为FooterView.而setLoadMoreView方法则是为该LoadMoreView设置监听，其接口如下：
>

    public interface LoadMoreView {

        /**
         * Show progress.
         */
        void onLoading();

        /**
         * Load finish, handle result.
         */
        void onLoadFinish(boolean dataEmpty, boolean hasMore);

        /**
         * Non-auto-loading mode, you can to click on the item to load.
         */
        void onWaitToLoadMore(LoadMoreListener loadMoreListener);

        /**
         * Load error.
         */
        void onLoadError(int errorCode, String errorMessage);
    }

自定义LoadMoreView时继承SwipeMenuRecyclerView.LoadMoreView并实现以上四个方法，我们以DefaultLoadMoreView为例：
>

    @Override
    public void onLoading() {
        setVisibility(VISIBLE);
        mLoadingView.setVisibility(VISIBLE);
        mTvMessage.setVisibility(VISIBLE);
        mTvMessage.setText(R.string.recycler_swipe_load_more_message);
    }

    @Override
    public void onLoadFinish(boolean dataEmpty, boolean hasMore) {
        if (!hasMore) {
            setVisibility(VISIBLE);

            if (dataEmpty) {
                mLoadingView.setVisibility(GONE);
                mTvMessage.setVisibility(VISIBLE);
                mTvMessage.setText(R.string.recycler_swipe_data_empty);
            } else {
                mLoadingView.setVisibility(GONE);
                mTvMessage.setVisibility(VISIBLE);
                mTvMessage.setText(R.string.recycler_swipe_more_not);
            }
        } else {
            setVisibility(INVISIBLE);
        }
    }

    @Override
    public void onWaitToLoadMore(SwipeMenuRecyclerView.LoadMoreListener loadMoreListener) {
        this.mLoadMoreListener = loadMoreListener;

        setVisibility(VISIBLE);
        mLoadingView.setVisibility(GONE);
        mTvMessage.setVisibility(VISIBLE);
        mTvMessage.setText(R.string.recycler_swipe_click_load_more);
    }

    @Override
    public void onLoadError(int errorCode, String errorMessage) {
        setVisibility(VISIBLE);
        mLoadingView.setVisibility(GONE);
        mTvMessage.setVisibility(VISIBLE);
        mTvMessage.setText(TextUtils.isEmpty(errorMessage) ? getContext().getString(R.string.recycler_swipe_load_error) : errorMessage);
    }

从上面看当加载完成时，判断是否还有更多数据，若没有隐藏LoadMoreView,若存在，则根据此次数据是否为空，显示不同的UI，是不是很熟悉？对了，这就解释了上面为什么最后一定要调用loadMoreFinish，因为loadMoreFinish方法中最终调用了onLoadFinish(dataEmpty, hasMore)。
>

    public final void loadMoreFinish(boolean dataEmpty, boolean hasMore) {
        isLoadMore = false;
        isLoadError = false;

        mDataEmpty = dataEmpty;
        mHasMore = hasMore;

        if (mLoadMoreView != null) {
            mLoadMoreView.onLoadFinish(dataEmpty, hasMore);
        }
    }

回到正题，接下来就是设置加载更多的监听，其接口为：
>

    public interface LoadMoreListener {

        /**
         * More data should be requested.
         */
        void onLoadMore();
    }

对，就是我们new出来的LoadMoreListener并实现了其onLoadMore方法。那么onLoadMore在哪被调用了，最终我们锁定了SwipeMuneRecyclerView中的dispatch-LoadMore
>

    private void dispatchLoadMore() {
        if (isLoadError) return;

        if (!isAutoLoadMore) {
            if (mLoadMoreView != null)
                mLoadMoreView.onWaitToLoadMore(mLoadMoreListener);
        } else {
            if (isLoadMore || mDataEmpty || !mHasMore) return;

            isLoadMore = true;

            if (mLoadMoreView != null)
                mLoadMoreView.onLoading();

            if (mLoadMoreListener != null)
                mLoadMoreListener.onLoadMore();
        }
    }

逻辑也很简单，判断是否为自动加载，若不是，调用onWaitToLoadMore，翻看前面的代码可以看到其中传入了SwipeMenuRecyclerView.LoadMoreListener，在该
LoadMoreView的onClick中调用了mLoadMoreListener.onLoadMore()，即点击加载更多；若是，则判断是否正在加载，是否本次数据为空，是否没有更多数据，若都
不是，才执行mLoadMoreListener.onLoadMore()。
SwipeMenuRecyclerView继承自RecyclerView，而RecyclerView中又存在抽象类OnScrollListener，这里重写的就是其中的onScrollStateChanged和onScrolled
方法。onScrollStateChanged表示滑动状态变化时调用，onScrolled则表示滑动时不停的调用。
>

    @Override
    public void onScrollStateChanged(int state) {
        this.mScrollState = state;
    }

    @Override
    public void onScrolled(int dx, int dy) {
        LayoutManager layoutManager = getLayoutManager();
        if (layoutManager != null && layoutManager instanceof LinearLayoutManager) {
            LinearLayoutManager linearLayoutManager = (LinearLayoutManager) layoutManager;

            int itemCount = layoutManager.getItemCount();
            if (itemCount <= 0) return;

            int lastVisiblePosition = linearLayoutManager.findLastVisibleItemPosition();

            if (itemCount == lastVisiblePosition + 1 &&
                    (mScrollState == SCROLL_STATE_DRAGGING || mScrollState == SCROLL_STATE_SETTLING)) {
                dispatchLoadMore();
            }
        } else if (layoutManager != null && layoutManager instanceof StaggeredGridLayoutManager) {
            StaggeredGridLayoutManager staggeredGridLayoutManager = (StaggeredGridLayoutManager) layoutManager;

            int itemCount = layoutManager.getItemCount();
            if (itemCount <= 0) return;

            int[] lastVisiblePositionArray = staggeredGridLayoutManager.findLastCompletelyVisibleItemPositions(null);
            int lastVisiblePosition = lastVisiblePositionArray[lastVisiblePositionArray.length - 1];

            if (itemCount == lastVisiblePosition + 1 &&
                    (mScrollState == SCROLL_STATE_DRAGGING || mScrollState == SCROLL_STATE_SETTLING)) {
                dispatchLoadMore();
            }
        }
    }

从上面onScrolled中代码看出，当滑动时判断布局，并获取最后的位置，判断当滑动到最后位置，滑动状态为正在拖拽或者拖拽结束时，调用dispatchLoadMore方法实
现加载更多。


