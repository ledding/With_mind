# **SwipeRecyclerView学习之路（二）**
#### _本着初学者的精神，哪怕再简单的东西都要记下_

概述
--------------------
在上一篇文章中简单的介绍了SwipeRecyclerView的列表展示（其实就是RecyclerView的使用）和侧滑菜单，本篇我们就来分析分析侧滑菜单的实现


废话不多说 肝
--------------------
从哪开始说起呢，嗯，就从setAdapter开始吧，SwipeMenuRecyclerView不仅继承了RecyclerView还重写了其中的setAdapter.  
> 

    @Override
    public void setAdapter(Adapter adapter) {
        if (mAdapterWrapper != null) {
            mAdapterWrapper.getOriginAdapter().unregisterAdapterDataObserver(mAdapterDataObserver);
        }

        if (adapter == null) {
            mAdapterWrapper = null;
        } else {
            adapter.registerAdapterDataObserver(mAdapterDataObserver);

            mAdapterWrapper = new SwipeAdapterWrapper(getContext(), adapter);
            mAdapterWrapper.setSwipeItemClickListener(mSwipeItemClickListener);
            mAdapterWrapper.setSwipeMenuCreator(mSwipeMenuCreator);
            mAdapterWrapper.setSwipeMenuItemClickListener(mSwipeMenuItemClickListener);  
        ......  
        }
        super.setAdapter(mAdapterWrapper);
    }  
    
我们重写的setAdapter其实偷偷的做了很多事情，首先订阅了adapterDataObserver，这是一种观察者模式，其次创建了SwipeAdapterWrapper类、设置了列表项点击监听、设置了侧滑菜单创建器、设置了侧滑菜单列表项点击监听。其中侧滑菜单的布局、HeaderView和FootView都是通过SwipeAdapterWrapper类控制的。  
> 

    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        if (mHeaderViews.get(viewType) != null) {
            return new ViewHolder(mHeaderViews.get(viewType));
        } else if (mFootViews.get(viewType) != null) {
            return new ViewHolder(mFootViews.get(viewType));
        }
        final RecyclerView.ViewHolder viewHolder = mAdapter.onCreateViewHolder(parent, viewType);

        if (mSwipeItemClickListener != null) {
            viewHolder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mSwipeItemClickListener.onItemClick(v, viewHolder.getAdapterPosition());
                }
            });
        }

        if (mSwipeMenuCreator == null) return viewHolder;

        final SwipeMenuLayout swipeMenuLayout = (SwipeMenuLayout) mInflater.inflate(R.layout.recycler_swipe_view_item, parent, false);
        SwipeMenu swipeLeftMenu = new SwipeMenu(swipeMenuLayout, viewType);
        SwipeMenu swipeRightMenu = new SwipeMenu(swipeMenuLayout, viewType);

        mSwipeMenuCreator.onCreateMenu(swipeLeftMenu, swipeRightMenu, viewType);

        int leftMenuCount = swipeLeftMenu.getMenuItems().size();
        if (leftMenuCount > 0) {
            SwipeMenuView swipeLeftMenuView = (SwipeMenuView) swipeMenuLayout.findViewById(R.id.swipe_left);
            // noinspection WrongConstant
            swipeLeftMenuView.setOrientation(swipeLeftMenu.getOrientation());
            swipeLeftMenuView.createMenu(swipeLeftMenu, swipeMenuLayout, mSwipeMenuItemClickListener, SwipeMenuRecyclerView.LEFT_DIRECTION);
        }

        int rightMenuCount = swipeRightMenu.getMenuItems().size();
        if (rightMenuCount > 0) {
            SwipeMenuView swipeRightMenuView = (SwipeMenuView) swipeMenuLayout.findViewById(R.id.swipe_right);
            // noinspection WrongConstant
            swipeRightMenuView.setOrientation(swipeRightMenu.getOrientation());
            swipeRightMenuView.createMenu(swipeRightMenu, swipeMenuLayout, mSwipeMenuItemClickListener, SwipeMenuRecyclerView.RIGHT_DIRECTION);
        }

        ViewGroup viewGroup = (ViewGroup) swipeMenuLayout.findViewById(R.id.swipe_content);
        viewGroup.addView(viewHolder.itemView);

        try {
            Field itemView = getSupperClass(viewHolder.getClass()).getDeclaredField("itemView");
            if (!itemView.isAccessible()) itemView.setAccessible(true);
            itemView.set(viewHolder, swipeMenuLayout);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return viewHolder;
    }  

可以看到，作者在这里创建了SwipeMenu的两个实例，分别用作左滑菜单和右滑菜单，并调用onCreateMenu创建，所以在activity或fragment中setSwipemenuCreate必须在setAdapter之前。回到这里，当MenuItem集合有数据时，创建SwipeMenuView实例并调用其createMenu方法创建菜单,SwipeMenuView是自定义控件，继承了LinearLayout。  
> 

    public void createMenu(SwipeMenu swipeMenu, SwipeSwitch swipeSwitch,
                           SwipeMenuItemClickListener swipeMenuItemClickListener,
                           @SwipeMenuRecyclerView.DirectionMode int direction) {
        removeAllViews();

        this.mSwipeSwitch = swipeSwitch;
        this.mItemClickListener = swipeMenuItemClickListener;
        this.mDirection = direction;

        List<SwipeMenuItem> items = swipeMenu.getMenuItems();
        for (int i = 0; i < items.size(); i++) {
            SwipeMenuItem item = items.get(i);

            LayoutParams params = new LayoutParams(item.getWidth(), item.getHeight());
            params.weight = item.getWeight();
            LinearLayout parent = new LinearLayout(getContext());
            parent.setId(i);
            parent.setGravity(Gravity.CENTER);
            parent.setOrientation(VERTICAL);
            parent.setLayoutParams(params);
            ViewCompat.setBackground(parent, item.getBackground());
            parent.setOnClickListener(this);
            addView(parent);

            SwipeMenuBridge menuBridge = new SwipeMenuBridge(mDirection, i, mSwipeSwitch, parent);
            parent.setTag(menuBridge);

            if (item.getImage() != null) {
                ImageView iv = createIcon(item);
                menuBridge.mImageView = iv;
                parent.addView(iv);
            }

            if (!TextUtils.isEmpty(item.getText())) {
                TextView tv = createTitle(item);
                menuBridge.mTextView = tv;
                parent.addView(tv);
            }
        }
    }

菜单集合中获取列表，for循环设置属性样式并调用addView添加到父类LinnearLayout中去，这里的列表集合即为我们在onCreateMenu中创建的SwipeMenuItem的集合，这里的属性样式即为我们在onCreateMenu设置SwipeMenuItem的属性样式。  
> 

    private SwipeMenuCreator mSwipeMenuCreator = new SwipeMenuCreator() {
        @Override
        public void onCreateMenu(SwipeMenu swipeLeftMenu, SwipeMenu swipeRightMenu, int viewType) {
            int width = getResources().getDimensionPixelSize(R.dimen.dp_70);

            // 1. MATCH_PARENT 自适应高度，保持和Item一样高;
            // 2. 指定具体的高，比如80;
            // 3. WRAP_CONTENT，自身高度，不推荐;
            int height = ViewGroup.LayoutParams.MATCH_PARENT;

            SwipeMenuItem addItem = new SwipeMenuItem(getContext())
                    .setBackground(R.drawable.selector_green)
                    .setImage(R.mipmap.ic_action_add)
                    .setWidth(width)
                    .setHeight(height);
            swipeLeftMenu.addMenuItem(addItem); // 添加菜单到左侧。

            SwipeMenuItem closeItem = new SwipeMenuItem(getContext())
                    .setBackground(R.drawable.selector_green)
                    .setImage(R.mipmap.ic_action_close)
                    .setWidth(width)
                    .setHeight(height);
            swipeRightMenu.addMenuItem(closeItem); // 添加菜单到右侧。
        }
    };

在onCreateMenu方法中存在3个参数，swipeMenu,SwipeMenu和ViewType,其中前两个SwipeMenu就是我们在上面说到的SwipeAdapterWrapper中创建的左滑菜单和右滑菜单，下面贴出SwipeMenu的部分代码  
> 

    @IntDef({HORIZONTAL, VERTICAL})
    @Retention(RetentionPolicy.SOURCE)
    public @interface OrientationMode {
    }

    public static final int HORIZONTAL = LinearLayout.HORIZONTAL;
    public static final int VERTICAL = LinearLayout.VERTICAL;
    private SwipeMenuLayout mSwipeMenuLayout;

    private int mViewType;

    private int orientation = HORIZONTAL;
    private List<SwipeMenuItem> mSwipeMenuItems = new ArrayList<>(2);

    public SwipeMenu(SwipeMenuLayout swipeMenuLayout, int viewType) {
        this.mSwipeMenuLayout = swipeMenuLayout;
        this.mViewType = viewType;
    }   
    ......


其中的SwipeMenuLayout为侧滑菜单的布局方式，其继承自FramLayout并实现了SwipeSwitch方法，通过重写onTouchEvent实现手势侧滑，我们重点看一下onTouchEvent中的MotionEvent.ACTION_MOVE中的执行过程  
> 

            case MotionEvent.ACTION_MOVE: {
                if (!isSwipeEnable()) break;
                int disX = (int) (mLastX - ev.getX());
                int disY = (int) (mLastY - ev.getY());
                if (!mDragging && Math.abs(disX) > mScaledTouchSlop && Math.abs(disX) > Math.abs(disY)) {
                    mDragging = true;
                }
                if (mDragging) {
                    if (mSwipeCurrentHorizontal == null || shouldResetSwipe) {
                        if (disX < 0) {
                            if (mSwipeLeftHorizontal != null)
                                mSwipeCurrentHorizontal = mSwipeLeftHorizontal;
                            else
                                mSwipeCurrentHorizontal = mSwipeRightHorizontal;
                        } else {
                            if (mSwipeRightHorizontal != null)
                                mSwipeCurrentHorizontal = mSwipeRightHorizontal;
                            else
                                mSwipeCurrentHorizontal = mSwipeLeftHorizontal;
                        }
                    }
                    scrollBy(disX, 0);
                    mLastX = (int) ev.getX();
                    mLastY = (int) ev.getY();
                    shouldResetSwipe = false;
                }
                break;
            }  
            
计算x,y方向上手势的相对移动距离，当水平方向上的移动距离大于垂直方向上且水平移动距离大于缩放手势偏颇时，执行scrollBy(disX,0)水平滑动，并根据水平方向上的正负值判断左滑还是右滑。自此，SwipeMenuRecyclerView的侧滑菜单实现的粗略分析完了。
