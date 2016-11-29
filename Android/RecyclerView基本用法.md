---
title: RecyclerView的介绍与基本用法
date: 2016-10-25
tags: [Android, Recyclerview]
categories: Android
description:
---
RecyclerView的介绍与基本用法
<!--more-->

RecyclerView是Google在发布Android L时在support-v7包中添加的一个新组件，旨在用来替代ListView。RecyclerView的灵活性要比ListView更好。  
RecyclerView与ListView原理是类似的：都是仅仅维护少量的View并且可以展示大量的数据集。  
RecyclerView用以下两种方式简化了数据的展示和处理:  

+ 使用LayoutManager来确定每一个item的排列方式。
+ 为增加和删除项目提供默认的动画效果。

你也可以定义你自己的LayoutManager和添加删除动画，RecyclerView项目结构如下：   

+ Adapter：使用RecyclerView之前，你需要一个继承自RecyclerView.Adapter的适配器，作用是将数据与每一个item的界面进行绑定。
+ LayoutManager：用来确定每一个item如何进行排列摆放，何时展示和隐藏。回收或重用一个View的时候，LayoutManager会向适配器请求新的数据来替换旧的数据，这种机制避免了创建过多的View和频繁的调用findViewById方法（与ListView原理类似）。  
    目前SDK中提供了三种自带的LayoutManager:    
    + LinearLayoutManager：线性布局，可以实现横向或者纵向的滑动列表（类型ListVIew）
    + GridLayoutManager：表格布局，可以实现类似GridView布局
    + StaggeredGridLayoutManager：瀑布流布局，可以实现瀑布流效果

当然除了上面的三种内部布局之外，我们还可以继承RecyclerView.LayoutManager来实现一个自定义的LayoutManager。
Animations(动画)效果：RecyclerView对于Item的添加和删除是默认开启动画的。我们当然也可以通过RecyclerView.ItemAnimator类定制动画，然后通过RecyclerView.setItemAnimator()方法来进行使用。
RecyclerView相关的类：  

类                           |说明
-------                      |-----------
RecyclerView.Adapter         |托管数据集合，为每个Item创建视图
RecyclerView.ViewHolder      |承载Item视图的子视图
RecyclerView.LayoutManager   |负责Item视图的布局
RecyclerView.ItemDecoration  |为每个Item视图添加子视图，在Demo中被用来绘制Divider
RecyclerView.ItemAnimator    |负责添加、删除数据时的动画效果

## RecyclerView的基本用法
1、因为RecyclerView是support-v7中的控件，要使用的话我们首先要在build.gradle文件中添加RecyclerView的依赖，添加后我们就能在我们的项目中使用RecyclerView控件了：

```gradle
dependencies {
    ....
    compile 'com.android.support:recyclerview-v7:23.2.1'
}
```
2、在xml布局文件中使用RecyclerView：
```xml
<android.support.v7.widget.RecyclerView
    android:id="@+id/rv_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scrollbars="none"/>
```
3、对RecyclerView进行获取并设置：

```java
public class MainActivity extends AppCompatActivity {
    private RecyclerView mRecyclerView;
    private MyAdapter mAdapter;
    private ArrayList<String> mDatas = new ArrayList<>();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mRecyclerView = (RecyclerView) findViewById(R.id.rv_view);
        //设置LayoutManager类似ListView效果
        mRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity()));
        //类似横向ListView效果，不反转
        //mRecyclerView.setLayoutManager(new LinearLayoutManager(getActivity(), LinearLayoutManager.HORIZONTAL, false));
        //类似GridView效果，2列
        //mRecyclerView.setLayoutManager(new GridLayoutManager(getActivity(), 2));
        //竖向瀑布流效果，2列
        //mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(2, StaggeredGridLayoutManager.VERTICAL));
        mRecyclerView.setItemAnimator(new DefaultItemAnimator()); //设置默认动画
        for (int i = 0; i < 20; i++) {
            mDatas.add("卡片：" + i);
        }
        mAdapter = new MyAdapter(getActivity(), mDatas);
        mRecyclerView.setAdapter(mAdapter);
        mAdapter.setOnItemClickLitener(new MyAdapter.OnItemClickLitener() {
            @Override
            public void onItemClick(View view, int position) {
                Toast.makeText(getActivity(), "卡片" + position, Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```
Adapter:  

```java
public class MyAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder> implements View.OnClickListener {

    private LayoutInflater mInflater;
    private ArrayList<String> mDatas = new ArrayList<>();
    private static final int ZERO = 0;
    private static final int ONE = 1;
    private OnItemClickLitener mOnItemClickLitener;

    public MyAdapter(Context context, ArrayList<String> datas) {
        this.mDatas = datas;
        this.mInflater = LayoutInflater.from(context);
    }

    //添加数据
    public void addItem(String model, int position) {
        mDatas.add(position, model);
        notifyItemInserted(position);
    }

    //更新数据
    public void setData(ArrayList<String> data) {
        this.mDatas= data;
    }

    //删除数据
    public void removeItem(String model) {
        int position = mDatas.indexOf(model);
        mDatas.remove(position);
        notifyItemRemoved(position);
    }

    //删除数据
    public void removeItem(int position) {
        mDatas.remove(position);
        notifyItemRemoved(position);
    }
    @Override
    public void onClick(View view) {
        if (mOnItemClickLitener != null) {
            mOnItemClickLitener.onItemClick(view, (int)view.getTag());
        }
    }

    public interface OnItemClickLitener {
        void onItemClick(View view, int position);
    }

    public void setOnItemClickLitener(OnItemClickLitener mOnItemClickLitener) {
        this.mOnItemClickLitener = mOnItemClickLitener;
    }


    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        if (viewType == ONE) {
            return new MyViewHolder(mInflater.inflate(R.layout.item_0, parent, false));
        } else {
            return new MyViewHolderTwo(mInflater.inflate(R.layout.item_1, parent, false));
        }
    }

    @Override
    public void onBindViewHolder(final RecyclerView.ViewHolder holder, final int position) {
        if (holder instanceof MyViewHolder) {
            MyViewHolder holder1 = (MyViewHolder) holder;
            holder1.txt.setText(mDatas.get(position));
        } else {
            MyViewHolderTwo holder1 = (MyViewHolderTwo) holder;
            holder1.txt.setText(mDatas.get(position));
        }
        if (mOnItemClickLitener != null) {
            holder.itemView.setOnClickListener(this);
            holder.itemView.setTag(position);
        }
    }

    @Override
    public int getItemViewType(int position) {
        if (position % 2 == 0) {
            return ZERO;
        } else {
            return ONE;
        }
    }

    @Override
    public int getItemCount() {
        return mDatas.size();
    }

    class MyViewHolder extends RecyclerView.ViewHolder {
        TextView txt;

        public MyViewHolder(View itemView) {
            super(itemView);
            txt = (TextView) itemView.findViewById(R.id.tv_txt);
        }
    }

    class MyViewHolderTwo extends RecyclerView.ViewHolder {
        TextView txt;

        public MyViewHolderTwo(View itemView) {
            super(itemView);
            txt = (TextView) itemView.findViewById(R.id.tv_txt);
        }
    }
}
```
因为RecyclerView默认没有实现Item的点击事件，所以Item的点击事件需要我们自己来设置，在上面的代码中我们也可以看出来：

```java
private OnItemClickLitener mOnItemClickLitener;

public interface OnItemClickLitener {
    void onItemClick(View view, int position);
}

public void setOnItemClickLitener(OnItemClickLitener mOnItemClickLitener) {
    this.mOnItemClickLitener = mOnItemClickLitener;
}

@Override
public void onClick(View view) {
    if (mOnItemClickLitener != null) {
        mOnItemClickLitener.onItemClick(view, (int)view.getTag());
    }
}

@Override
public void onBindViewHolder(final RecyclerView.ViewHolder holder, final int position) {
    if (holder instanceof MyViewHolder) {
        MyViewHolder holder1 = (MyViewHolder) holder;
        holder1.txt.setText(mDatas.get(position));
    } else {
        MyViewHolderTwo holder1 = (MyViewHolderTwo) holder;
        holder1.txt.setText(mDatas.get(position));
    }
    if (mOnItemClickLitener != null) {
        holder.itemView.setOnClickListener(this);
        holder.itemView.setTag(position);
    }
}
```
```java
mAdapter.setOnItemClickLitener(new MyAdapter.OnItemClickLitener() {
    @Override
    public void onItemClick(View view, int position) {
        Toast.makeText(getActivity(), "卡片" + position, Toast.LENGTH_SHORT).show();
    }
});
```
注意：
在ListView当中，修改数据后可以用Adapter的notifyDataSetChanged()更新界面。而在RecyclerView中还有一些别的方法可以更新数据和界面：  
```java
//添加数据
public void addItem(String model, int position) {
    mDatas.add(position, model);
    notifyItemInserted(position);
}

//添加数据集
public void setData(ArrayList<String> data) {
    this.mDatas= data;
    notifyDataSetChanged();
}

//删除数据
public void removeItem(String model) {
    int position = mDatas.indexOf(model);
    mDatas.remove(position);
    notifyItemRemoved(position);
}

//删除数据
public void removeItem(int position) {
    mDatas.remove(position);
    notifyItemRemoved(position);
}
```
以上就是RecyclerView的基本用法。