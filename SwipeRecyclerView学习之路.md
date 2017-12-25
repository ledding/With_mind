# **SwipeRecyclerView学习之路**

#### _本着初学者的精神 哪怕在简单的东西都要记下_

## 简介
> SwipeRecyclerView是基于RecyclerView封装的github上一个经典的开源库，目前版本v1.1.0，具有侧滑菜单、拖拽排序、侧滑删除、添加或移除HeaderView和FooterView、自动或点击加载更多等特性，因此成为了android程序猿偏爱的列表控件。


## 详情
> *作者:* yanzhenjie(不认识？大神就是了)  
> *Github地址:* https://github.com/yanzhenjie/SwipeRecyclerView  
> *Star:* 截止目前2660，嗯、现在是2662了


## 使用
* ### 引入
----------------
	//SwipeRecyclerView
	compile 'com.yanzhenjie:recyclerview-swipe:1.1.3'


* ### 开发
------------------------------------------------------
#### 布局
	<com.yanzhenjie.recyclerview.swipe.SwipeMenuRecyclerView
	    android:id="@+id/swipe_rv"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	</com.yanzhenjie.recyclerview.swipe.SwipeMenuRecyclerView>

#### RecyclerView使用
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

