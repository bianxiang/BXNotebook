## 布局

### 概述

#### 编写XML

布局XML文件放在`res/layout/`目录。
布局文件只能有一个根元素，必须是View或ViewGroup。

例子：

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	              android:layout_width="fill_parent" 
	              android:layout_height="fill_parent" 
	              android:orientation="vertical" >
	    <TextView android:id="@+id/text"
	              android:layout_width="wrap_content"
	              android:layout_height="wrap_content"
	              android:text="Hello, I am a TextView" />
	    <Button android:id="@+id/button"
	            android:layout_width="wrap_content"
	            android:layout_height="wrap_content"
	            android:text="Hello, I am a Button" />
	</LinearLayout>

#### 加载XML资源

例如，如果XML布局是`main_layout.xml`，可以这样加载：

	public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.main_layout);
	}

#### 特性

一些特性是View特有的（如`TextView`支持`textSize`特性），一些是从父类继承的，一些是布局参数，是对象的父`ViewGroup`定义的。

##### ID

To uniquely identify the View within the tree.

	android:id="@+id/my_button"

The at-symbol (@) at the beginning of the string indicates that the XML parser should parse and expand the rest of the ID string and identify it as an ID resource. The plus-symbol (+) means that this is a new resource name that must be created and added to our resources (in the R.java file). 还有一些ID资源是Android框架提供的。引用Android的资源ID时，不需要用+号，但需要`android`前缀，如

	android:id="@android:id/empty"

With the android package namespace in place, we're now referencing an ID from the `android.R` resources class, rather than the local resources class.

ID不需要在整个树中唯一，只需要在你搜索的字数中唯一。但由于经常需要搜索整个树，因此还是完全唯一较好。

##### 布局参数

布局特性形如`layout_something`。

每个`ViewGroup`都实现了一个继承自`ViewGroup.LayoutParams`的嵌套类。The parent view group defines layout parameters for each child view (including the child view group).

Each child element must define LayoutParams that are appropriate for its parent, though it may also define different LayoutParams for its own children.

所有的ViewGroup都包含`layout_width`和`layout_height`参数，因此每个View都需要定义它们。

#### 布局位置

View的大小和位置的单位是像素。获取位置可以调用`getLeft()`和`getTop()`。位置相对于父。此外还有`getRight()`和`getBottom()`方法。

#### 大小、Padding 和 Margins

View有两种大小。

第一种叫做*measured width*和*measured height*。方法： `getMeasuredWidth()`、 `getMeasuredHeight()` 。These dimensions define how big a view wants to be within its parent. 

第二种叫做，*width*和*height*，或者叫`drawing width``drawing height`。方法：`getWidth()`和`getHeight()`。它们是布局后View在屏幕上的实际大小。

计算View大小时考虑padding。方法`setPadding(int, int, int, int)`、`getPaddingLeft()`、`getPaddingTop()`、`getPaddingRight()`、`getPaddingBottom()`。

View不支持Margin。但ViewGroup支持，参见[ViewGroup](http://developer.android.com/reference/android/view/ViewGroup.html)和[ViewGroup.MarginLayoutParams](http://developer.android.com/reference/android/view/ViewGroup.MarginLayoutParams.html)。

#### 适配器

If, during the course of your application's life, you change the underlying data that is read by your adapter, you should call `notifyDataSetChanged()`. This will notify the attached view that the data has been changed and it should refresh itself.

### （未）Linear Layout

### （未）Relative Layout

### （未）List View

### Grid View

GridView是一个二维的、可滚动的Grid。使用ListAdapter向布局中插入单元格。

![](http://developer.android.com/images/ui/gridview.png)

例子：显示图片缩略图网格。

	<?xml version="1.0" encoding="utf-8"?>
	<GridView xmlns:android="http://schemas.android.com/apk/res/android" 
	    android:id="@+id/gridview"
	    android:layout_width="fill_parent" 
	    android:layout_height="fill_parent"
	    android:columnWidth="90dp"
	    android:numColumns="auto_fit"
	    android:verticalSpacing="10dp"
	    android:horizontalSpacing="10dp"
	    android:stretchMode="columnWidth"
	    android:gravity="center" />

代码：

	public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
	    setContentView(R.layout.main);
	
	    GridView gridview = (GridView) findViewById(R.id.gridview);
	    gridview.setAdapter(new ImageAdapter(this));
	
	    gridview.setOnItemClickListener(new OnItemClickListener() {
	        public void onItemClick(AdapterView<?> parent, View v, int position, long id) {
	            Toast.makeText(HelloGridView.this, "" + position, Toast.LENGTH_SHORT).show();
	        }
	    });
	}

	public class ImageAdapter extends BaseAdapter {
	    private Context mContext;
	
	    public ImageAdapter(Context c) {
	        mContext = c;
	    }
	
	    public int getCount() {
	        return mThumbIds.length;
	    }
	
	    public Object getItem(int position) {
	        return null;
	    }
	
	    public long getItemId(int position) {
	        return 0;
	    }
	
	    // create a new ImageView for each item referenced by the Adapter
	    public View getView(int position, View convertView, ViewGroup parent) {
	        ImageView imageView;
	        if (convertView == null) {  // if it's not recycled, initialize some attributes
	            imageView = new ImageView(mContext);
	            imageView.setLayoutParams(new GridView.LayoutParams(85, 85));
	            imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
	            imageView.setPadding(8, 8, 8, 8);
	        } else {
	            imageView = (ImageView) convertView;
	        }
	
	        imageView.setImageResource(mThumbIds[position]);
	        return imageView;
	    }
	
	    // references to our images
	    private Integer[] mThumbIds = {
	            R.drawable.sample_2, R.drawable.sample_3,
	            R.drawable.sample_4, R.drawable.sample_5,
	            R.drawable.sample_6, R.drawable.sample_7,
	            R.drawable.sample_0, R.drawable.sample_1,
	            R.drawable.sample_2, R.drawable.sample_3,
	            R.drawable.sample_4, R.drawable.sample_5,
	            R.drawable.sample_6, R.drawable.sample_7,
	            R.drawable.sample_0, R.drawable.sample_1,
	            R.drawable.sample_2, R.drawable.sample_3,
	            R.drawable.sample_4, R.drawable.sample_5,
	            R.drawable.sample_6, R.drawable.sample_7
	    };
	}










