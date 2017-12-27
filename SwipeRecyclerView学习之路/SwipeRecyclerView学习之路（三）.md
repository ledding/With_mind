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
