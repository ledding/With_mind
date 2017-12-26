# **SwipeRecyclerView学习之路(一)**

#### _本着初学者的精神 哪怕在简单的东西都要记下_

## 简介
> SwipeRecyclerView是基于RecyclerView封装的github上一个经典的开源库，目前版本v1.1.0，具有侧滑菜单、拖拽排序、侧滑删除、添加或移除HeaderView和FooterView、自动或点击加载更多等特性，因此成为了android程序猿偏爱的列表控件。


## 详情
> *作者:* yanzhenjie(不认识？大神就是了)  
> *Github地址:* https://github.com/yanzhenjie/SwipeRecyclerView  
> *Star:* 截止目前2660，嗯、现在是2662了


## 使用
### 引入
----------------
	//SwipeRecyclerView
	compile 'com.yanzhenjie:recyclerview-swipe:1.1.3'


### 开发
------------------------------------------------------
* #### 布局  
> 
	<com.yanzhenjie.recyclerview.swipe.SwipeMenuRecyclerView
	    android:id="@+id/swipe_rv"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	</com.yanzhenjie.recyclerview.swipe.SwipeMenuRecyclerView>

* #### RecyclerView使用
1. 初始化数据。你可以从服务器获取、本地数据库拿，也可以自己zou  
> 
	private void setData() {  
		for (int i = 0; i < 13; i++) {  
			Book book = new Book();  
			book.setName("recycler item " + (i + 1));  
			books.add(book);  
		}  
	}

2. 在Activity或者Fragment 中初始化控件、并设置必要的layoutMnager  
> 
	swipeMenu_rv = (SwipeMenuRecyclerView) findViewById(R.id.swipe_rv); //初始化控件
	LinearLayoutManager layoutManager = new LinearLayoutManager(this); //线性
	layoutManager.setOrientation(LinearLayoutManager.VERTICAL);        //纵向排列
	swipeMenu_rv.setLayoutManager(layoutManager); //设置layoutManager
3. 创建你的adapter，在那之前呢，需要你创建item布局（这个看个人喜好，只要不偏离社会就好~·~），以下是adapter的代码  
> 
	public class SwipeMuneRVAdapter extends RecyclerView.Adapter<SwipeMuneRVAdapter.ViewHolder>{
	    private Context mContext;
	    private List<Book> data;
	
	    public SwipeMuneRVAdapter(Context mContext, List<Book> data) {
	        this.mContext = mContext;
 	       this.data = data;
	    }

	    public class ViewHolder extends RecyclerView.ViewHolder{
	        public TextView tvName;
	        private View view;
	        public ViewHolder(View itemView) {
	            super(itemView);
	            view = this.itemView;
	            tvName = (TextView) itemView.findViewById(R.id.recycler_item_tv_name);
	        }
	    }


	    @Override
	    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
	        View view = LayoutInflater.from(mContext).inflate(R.layout.swipe_menu_item,parent,false);
	        ViewHolder viewHolder = new ViewHolder(view);
	        return viewHolder;
	    }

	    @Override
	    public void onBindViewHolder(ViewHolder holder, int position) {
	        holder.tvName.setText(""+data.get(position).getName());
	    }

	    @Override
	    public int getItemCount() {
	        return data.size();
	    }
	}

4. 最后在回到Activity或Fragment中创建adapter，并设置  
> 
	recyclerAdapter = new RecyclerAdapter(this, books);
	swipeMenu_rv.setAdapter(recyclerAdapter);

* #### 侧滑菜单
1. 侧滑菜单集成的很方便，你只需要这样就行了  
> 
	swipeMenu_rv.setSwipeMenuCreator(mMenuCreator);
	
2. 那么mMenuCreator没有怎么办，没有当然是创建啦，只需要创建一个SwipeMenuCreator即可  
> 
	SwipeMenuCreator mMenuCreator = new SwipeMenuCreator() {
	            @Override
	            public void onCreateMenu(SwipeMenu swipeLeftMenu, SwipeMenu swipeRightMenu, int viewType) {
	            	//TODO 创建item菜单
	            }
	};
	
3. 创建item菜单并设置其属性样式  
> 
	SwipeMenuItem deleteMenu = new SwipeMenuItem(getApplicationContext());
	//添加菜单宽高、图片、背景色等
	deleteMenu.setWidth(200);
	deleteMenu.setHeight(LinearLayoutCompat.LayoutParams.MATCH_PARENT);
	deleteMenu.setImage(R.mipmap.delete);
	deleteMenu.setBackgroundColor(Color.RED);
	swipeRightMenu.addMenuItem(deleteMenu); //在item的右侧添加一个菜单

   由于作者在SwipeMenuItem类中set方法返回值都为SwipeMenuItem，所以我们也可以用链表的方式设置其属性样式
> 
	public SwipeMenuItem setBackground(@DrawableRes int resId) {
	    return setBackground(ContextCompat.getDrawable(mContext, resId));
	}

	public SwipeMenuItem setBackground(Drawable background) {
	    this.background = background;
	    return this;
	}

	SwipeMenuItem updateMenu = new SwipeMenuItem(getApplicationContext());
	//链式添加菜单属性内容
	updateMenu.setWidth(200)
	           .setHeight(LinearLayoutCompat.LayoutParams.MATCH_PARENT)
	           .setText("点击更改")
	           .setTextSize(22)
	           .setTextColor(Color.WHITE)
	           .setBackgroundColor(Color.parseColor("#aaaaaa"));
	swipeLeftMenu.addMenuItem(updateMenu); //在item的左侧添加一个菜单

4. 运行下代码，右滑一下，右滑菜单出来了，但点击一下却是没什么反应，当然没反应，你没设置item菜单点击事件  
> 
	//item菜单的点击监听
        SwipeMenuItemClickListener mMenuItemClickListener = new SwipeMenuItemClickListener() {
            @Override
            public void onItemClick(SwipeMenuBridge menuBridge) {
                //任何操作前，必须先关闭菜单，否则可能出现Item菜单打开状态错乱
                menuBridge.closeMenu();

                int direction = menuBridge.getDirection(); //左侧还是右侧菜单 左侧：1 右侧：-1;
                int adapterPosition = menuBridge.getAdapterPosition(); //item的position
                int menuPosition = menuBridge.getPosition(); //item中菜单的Position

                Toast.makeText(SwipeRVActivity.this, "direction:" + direction + "\nadpterPosition:" + adapterPosition + "\nmenuPosition:" + menuPosition, Toast.LENGTH_SHORT).show();

            }
        };
        swipeMenu_rv.setSwipeMenuItemClickListener(mMenuItemClickListener);
	
