# **SwipeRecyclerView学习之路（三）**
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
   
