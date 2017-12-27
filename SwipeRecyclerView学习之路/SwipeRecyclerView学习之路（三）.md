# **SwipeRecyclerView学习之路（三）**
#### *本着初学者的精神 哪怕再简单的东西都要记下*

概述
-------------------------------
前面我们介绍了SwipeRecyclerView特性之一侧滑菜单的使用和粗略的原理分析，接下来将继续往后学习。本篇主要介绍HeaderView和FooterView的运用与分析。


运用
-------------------------------
在SwipeRcyclerView中，作者对HeaderView和FooterView集成的很完美，使得运用起来简单方便，只需要调用addHeaderView(headerView)、addFooterView(footerView)来添加。当你不需要时，也可以调用removeHeaderView(headerView)、removeFooterView(footerView)来移除。  
> 

        //HeaderView和FooterView
        View headerView = LayoutInflater.from(this).inflate(R.layout.header_view,null);
        swipeMenu_rv.addHeaderView(headerView);
        swipeMenu_rv.removeView(headerView);

        View footerView = LayoutInflater.from(this).inflate(R.layout.footer_view,null);
        swipeMenu_rv.addFooterView(footerView);
        swipeMenu_rv.removeView(footerView);

值得注意的是，由于HeaderView占位的关系，如果添加了HeaderView，凡是通过ViewHolder拿到的position都要减掉HeaderView的数量才能得到正确的item position。所以应运而生了一下几个方法：  
>

        getHeaderItemCount(); // 获取HeaderView个数。
        getFooterItemCount(); // 获取FooterView个数。
        getItemViewType(int); // 获取Item的ViewType，包括HeaderView、FooterView、普通ItemView。


分析
--------------------------------
我们拿HeaderView举例，还是从setAdapter说起，在SwipeMenuRecyclerView有这么几行代码  
> 

            if (mHeaderViewList.size() > 0) {
                for (View view : mHeaderViewList) {
                    mAdapterWrapper.addHeaderView(view);
                }
            }

调用SwipeAdapterWrapper的addHeaderView方法，将集合中的HeaderView全部添加进去  
> 

    public void addHeaderView(View view) {
        mHeaderViews.put(getHeaderItemCount() + BASE_ITEM_TYPE_HEADER, view);
    }

这里要说一下，在SwipeAdapterWrapper中存亡headerView的集合为SparseArrayCompat，它是google推荐的一种Map容器，其使用了一套算法优化了hashMap,可以节省至少50%的缓存  
> 

    private SparseArrayCompat<View> mHeaderViews = new SparseArrayCompat<>();
    private SparseArrayCompat<View> mFootViews = new SparseArrayCompat<>();

那么HeaderView到底是如何适配的呢？前面我们说过SwipeAdapterWrapper其实就是自定义的RecyclerView.Adapter,在其重写的onCreateViewHolder中，我们可以看到
> 

        if (mHeaderViews.get(viewType) != null) {
            return new ViewHolder(mHeaderViews.get(viewType));
        } else if (mFootViews.get(viewType) != null) {
            return new ViewHolder(mFootViews.get(viewType));
        }

解释这个之前先要解释另一个重写的方法getItemViewType(int position)，不难看出就是根据下标获取viewType值  
> 

    @Override
    public int getItemViewType(int position) {
        if (isHeaderView(position)) {
            return mHeaderViews.keyAt(position);
        } else if (isFooterView(position)) {
            return mFootViews.keyAt(position - getHeaderItemCount() - getContentItemCount());
        }
        return mAdapter.getItemViewType(position - getHeaderItemCount());
    }

而viewType分为三类：headerView、footerView和中间的数据部分，addHeaderView中我们可以看到，其key值是在下标的基础上加了个常量BASE_ITEM_TYPE_HEADER，同理BASE_ITEM_TYPE_FOOTER也是。  
> 

    private static final int BASE_ITEM_TYPE_HEADER = 100000;
    private static final int BASE_ITEM_TYPE_FOOTER = 200000;

所以第一个headerView的viewType应该是100000，第二个是100001...回到前面那段没解释的代码，现在看来就理解了，通过传入的viewType判断headerView集合和FooterView集合该key所对应的View是否存在，如果存在new一个viewHolder传入该view并返回。
另外在activity中添加/移除HeaderView/FooterView喝setAdapter()的调用不分先后顺序，这是因为添加/移除HeaderView/FooterView最终调用的是SwipeAdapterWrapper中的addHeaderViewAndNotify、removeHeaderViewAndNotify，其在添加和移除数据的同时还通知了adapter更新UI。  
> 

        public void addHeaderViewAndNotify(View view) {
            mHeaderViews.put(getHeaderItemCount() + BASE_ITEM_TYPE_HEADER, view);
            notifyItemInserted(getHeaderItemCount() - 1);
        }

        public void removeHeaderViewAndNotify(View view) {
            int headerIndex = mHeaderViews.indexOfValue(view);
            mHeaderViews.removeAt(headerIndex);
            notifyItemRemoved(headerIndex);
        }
