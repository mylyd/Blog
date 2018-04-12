### 笔记主目录

#### FragmentViewPager简单Dome

### 1.构建适配器Adapter

```java
	/*注意打包是用support.v4的*/
public class FragmentViewPagerAdpater extends FragmentPagerAdapter{
    private List<Fragment> mListfragment; //创建集合类
    
    /*必须重写的三个方法*/
    
    //这里申请了一个Fragment的List对象，用于保存用于滑动的Fragment对象，并在创造函数中初始化：
   
    public FragmentViewPagerAdapter(FragmentManager fm,list<Fragment> mfragment){
        super(fm);
        this.mListfragment = mfragment;
    }
    
    //在getItem(int position)中，根据传来的参数arg0，来返回当前要显示的fragment
    public Fragment getItem(int position){
        return mListfragment.get(position);
    }
    
    //判断整个ViewPager的长度
    public int getCount(){
        return mListFragment.size();
    }
}
```

*注意*  <每个页面的相应功能均在对应的Fragment页面里面书写>

### 2.主要的 FragmentActivity <设置类的代码全写在这里面-->"VPActivity">

```java
/*注意打包是用support.v4的*/
public class VPActivity extends FragmentActivity implements View.OnClickListener {
    private List<Fragment> mList;
    private ViewPager vp;
    private TextView mText1;
    private TextView mText2;
    private TextView mText3;
    private TextView mText4;
    private TextView mText5;
    private TextView mView5;
    private TextView mView4;
    private TextView mView3;
    private TextView mView2;
    private TextView mView1;
    private int index;//当前页

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_view_pager);
        initView();
        setViewPager();
    }
    /* 初始化ViewPager*/
    private void setViewPager() {
        mList = new ArrayList<Fragment>();
        mList.add(new Fragment1());
        mList.add(new Fragment2());
        mList.add(new Fragment3());
        mList.add(new Fragment4());
        mList.add(new Fragment5());
        //初始化适配器
        FragmentManager fm = getSupportFragmentManager();
        ViewPagerAdapter ad = new ViewPagerAdapter(fm, mList);
        vp.setAdapter(ad);
        /*这里初始化页面写在了自定义的一个方法里面*/
        setTextColor(mText1, 0,mView1);//进入显示的第一页
		//对ViewPager进行监听，
        vp.addOnPageChangeListener(new ViewPager.OnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
            }
            /* 在这个方法中写对每个页面的逻辑 */
            @Override
            public void onPageSelected(int position) {
                index = position;
                /*position的初始值从0开始，即第一个页面为：0，...依次类推  */
                if (index == 0) {
                    setTextColor(mText1, 0,mView1);
                }
                if (index == 1) {
                    setTextColor(mText2, 1,mView2);
                }
                if (index == 2) {
                    setTextColor(mText3, 2,mView3);
                }
                if (index == 3) {
                    setTextColor(mText4, 3,mView4);
                }
                if (index == 4) {
                    setTextColor(mText5, 4,mView5);
                }
            }

            @Override
            public void onPageScrollStateChanged(int state) {
            }
        });
    }

    private void initView() {
        mText1 = (TextView) findViewById(R.id.fragment_text_1);
        mText1.setOnClickListener(this);
        mText2 = (TextView) findViewById(R.id.fragment_text_2);
        mText2.setOnClickListener(this);
        mText3 = (TextView) findViewById(R.id.fragment_text_3);
        mText3.setOnClickListener(this);
        mText4 = (TextView) findViewById(R.id.fragment_text_4);
        mText4.setOnClickListener(this);
        mText5 = (TextView) findViewById(R.id.fragment_text_5);
        mText5.setOnClickListener(this);
        mView1 = (TextView) findViewById(R.id.view_1);
        mView2 = (TextView) findViewById(R.id.view_2);
        mView3 = (TextView) findViewById(R.id.view_3);
        mView4 = (TextView) findViewById(R.id.view_4);
        mView5 = (TextView) findViewById(R.id.view_5);
        vp = (ViewPager) findViewById(R.id.view_pager);//ViewPager事件进行绑定
    }
	
    /* 单独点击视图上方的TextVIew进行不同ViewPager页面之间的跳转 */
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.fragment_text_1:
                // TODO 18/03/23
                setTextColor(mText1, 0,mView1);
                break;
            case R.id.fragment_text_2:
                // TODO 18/03/23
                setTextColor(mText2, 1,mView2);
                break;
            case R.id.fragment_text_3:
                // TODO 18/03/23
                setTextColor(mText3, 2,mView3);
                break;
            case R.id.fragment_text_4:
                // TODO 18/03/23
                setTextColor(mText4, 3,mView4);
                break;
            case R.id.fragment_text_5:
                // TODO 18/03/23
                setTextColor(mText5, 4,mView5);
                break;
            default:
                break;
        }
    }
	/**这个方法属于自定义 
		1.第一个参数代表页面之间显示的TextView；
		2.第二个参数代表显示页面对应的position值；
		3.第三个参数代表Title下方小视图的TextView；
    */
    private void setTextColor(TextView tv, int item,TextView v) {
        /* 将每个页面显示TextView设置为相同的颜色 */
        mText1.setTextColor(ContextCompat.getColor(this, R.color.PagerTitleText_off));
        mText2.setTextColor(ContextCompat.getColor(this, R.color.PagerTitleText_off));
        mText3.setTextColor(ContextCompat.getColor(this, R.color.PagerTitleText_off));
        mText4.setTextColor(ContextCompat.getColor(this, R.color.PagerTitleText_off));
        mText5.setTextColor(ContextCompat.getColor(this, R.color.PagerTitleText_off));
        /* 将每个页面Title视图的TextView设置为相同的颜色 */
        mView1.setBackgroundResource(R.color.PagerTitleView_off);
        mView2.setBackgroundResource(R.color.PagerTitleView_off);
        mView3.setBackgroundResource(R.color.PagerTitleView_off);
        mView4.setBackgroundResource(R.color.PagerTitleView_off);
        mView5.setBackgroundResource(R.color.PagerTitleView_off);
        //将每一个点击的<或者说是当前显示的页面设置独立的颜色以便区分>
        tv.setTextColor(ContextCompat.getColor(this, R.color.PagerTitleText_on));
        v.setBackgroundResource(R.color.PagerTitleView_on);
    	//对应position值的页面；
        vp.setCurrentItem(item);
    }
}
```

### 3.VPActivity的layout页面 

### 		< activity_view_pager.xml >

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <include layout="@layout/item"/>	<!-- 导入的Title部分 -->
    <android.support.v4.view.ViewPager
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/view_pager"/>
</LinearLayout>  

```

### 	< item.xml >

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="68dp"
        android:orientation="vertical">
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:gravity="center"
            android:background="#d2ede3"
            android:orientation="horizontal">
        <TextView
            android:id="@+id/fragment_text_1"
            android:layout_width="100dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="页面1"
            android:textColor="@color/PagerTitleText_off"
            android:layout_margin="10dp"
            android:textSize="36sp"/>
        <TextView
            android:id="@+id/fragment_text_2"
            android:layout_width="100dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="页面2"
            android:textColor="@color/PagerTitleText_off"
            android:layout_margin="10dp"
            android:textSize="36sp"/>
        <TextView
            android:id="@+id/fragment_text_3"
            android:layout_width="100dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="页面3"
            android:textColor="@color/PagerTitleText_off"
            android:layout_margin="10dp"
            android:textSize="36sp"/>
        <TextView
            android:id="@+id/fragment_text_4"
            android:layout_width="100dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="页面4"
            android:textColor="@color/PagerTitleText_off"
            android:layout_margin="10dp"
            android:textSize="36sp"/>
        <TextView
            android:id="@+id/fragment_text_5"
            android:layout_width="100dp"
            android:layout_height="wrap_content"
            android:layout_gravity="center"
            android:gravity="center"
            android:text="页面5"
            android:textColor="@color/PagerTitleText_off"
            android:layout_margin="10dp"
            android:textSize="36sp"/>
        </LinearLayout>
        <LinearLayout
            android:background="#d2ede3"
            android:layout_width="match_parent"
            android:orientation="horizontal"
            android:gravity="center"
            android:layout_height="4dp">
                <TextView
                   android:id="@+id/view_1"
                   android:layout_width="60dp"
                   android:layout_marginRight="30dp"
                   android:layout_marginLeft="25dp"
                   android:layout_height="match_parent"
                   android:background="@color/PagerTitleView_off"/>
                <TextView
                    android:id="@+id/view_2"
                    android:layout_width="60dp"
                    android:layout_marginRight="30dp"
                    android:layout_marginLeft="30dp"
                    android:layout_height="match_parent"
                    android:background="@color/PagerTitleView_off"/>
                <TextView
                    android:id="@+id/view_3"
                    android:layout_width="60dp"
                    android:layout_marginRight="30dp"
                    android:layout_marginLeft="30dp"
                    android:layout_height="match_parent"
                    android:background="@color/PagerTitleView_off"/>
                <TextView
                    android:id="@+id/view_4"
                    android:layout_width="60dp"
                    android:layout_marginRight="30dp"
                    android:layout_marginLeft="30dp"
                    android:layout_height="match_parent"
                    android:background="@color/PagerTitleView_off"/>
                <TextView
                    android:id="@+id/view_5"
                    android:layout_width="60dp"
                    android:layout_marginRight="30dp"
                    android:layout_marginLeft="30dp"
                    android:layout_height="match_parent"
                    android:background="@color/PagerTitleView_off"/>
        </LinearLayout>
</LinearLayout>
```

### 	colors.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#3F51B5</color>
    <color name="colorPrimaryDark">#303F9F</color>
    <color name="colorAccent">#FF4081</color>
    <!-- 以下是添加的 -->
    <color name="PagerTitleText_off">#aa848484</color>
    <color name="PagerTitleText_on">#aa39e182</color>
    <color name="PagerTitleView_off">#d2ede3</color>
    <color name="PagerTitleView_on">#0082fc</color>
</resources>

```

### 4.Fragment部分 < Fragment1 >(其余的页面相同 ....依次类推)

==< 每个ViewPager对应页面的功能在对应的Fragment里面写 >==

```java
public class Fragment1 extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater,ViewGroup container,Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.activity_fragment1,container,false);
        return view;
    }
}
```

### Fragment对应的layout < 随便写一点，能区分就行  .....后面的页面依次类推>

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:background="#0f8cb3"
    android:orientation="vertical">
    <TextView
        android:id="@+id/time_text"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:gravity="center"
        android:textSize="66dp"
        android:text="页面1"/>
</LinearLayout>  

```



### 实现ViewPager底部小圆点之间的切换

< 结合以上的代码 >

1. xml制作小点开始

   ```xml
   <!--//这里的文件建立在drawable里面//-->
   <!--dot_sty_b//这是"选中"小点样式的制作-->
   ```

   ​

