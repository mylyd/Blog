实用View 小功能 （一）———实现页面底部弹出式App评分引导功能

一、概述 

​		项目中经常需要实现底部弹出框这样的需求，实现方式有很多种，比如：Dialog、全局Activity、PopupWindow、DialogFragment都可以达到目的。下面主要讲解各种的大致用法，因为我使用的是Dialog和PopuoWindow我会详细讲解。

二、具体方式实现方案

通用的animation文件 （这两个xml需要放在res/anim的anim文件夹下）

  ``` xml
//进入动画     <具体实现看各种需求实现动画效果，这是一种常见的平移动画>
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="250"
        android:fromYDelta="100%p"
        android:toYDelta="0" />
    <alpha
        android:duration="250"
        android:fromAlpha="0.0"
        android:toAlpha="1.0" />
</set>
//退出动画
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="250"
        android:fromYDelta="0"
        android:toYDelta="50%p" />
    <alpha
        android:duration="250"
        android:fromAlpha="1.0"
        android:toAlpha="0.0" />
</set>
//注意：动画的进入与退出今年匹配成一套，这样看上去才会流畅不会出现直接跳动现象
  ```



1.全局Activity实现

a.创建一个普通常用的Activity文件 继承 Activity

```java
public class ActivityDialog extends Activity {
        @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_custom_pop_window);
        Window window = this.getWindow();
        //设置弹出位置
        window.setGravity(Gravity.BOTTOM);
        //设置弹出动画，如果在style里面设置过此处不用设置
        window.setWindowAnimations(R.style.bottom_dialog_anim_style);
        //设置透明度 ，如果在style里面设置过此处不用设置
        window.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        //设置dialog横向占比为全屏 ，通俗点就是：窗口显示的时候会出现悬浮在屏幕中间的情况，这里是设置让弹窗宽自动适应屏幕的宽度并且全部填充
        window.setLayout(WindowManager.LayoutParams.MATCH_PARENT, WindowManager.LayoutParams.WRAP_CONTENT);
        //选择一个定义在窗口中的View,在OnClick事件时响应退出动画效果，具体实现看个人需求而定
        findViewById(R.id.xxxxxx).setOnClickListener(v -> overridePendingTransition(0, R.anim.xxxxx));
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //在onTouchEvent中设置弹出时的动画效果
        finish();
        this.overridePendingTransition(0,R.anim.popshow);
        return true;
    }
}
```

b.在res/value/style中配置对应activity的属性

```xml
//设置动画效果，进入退出时的具体实现效果
    <style name="AnimBottom" parent="@android:style/Animation">
        <item name="android:windowEnterAnimation">@anim/show_in</item>  //进入动画设置
        <item name="android:windowExitAnimation">@anim/show_out</item>  //退出动画设置
    </style>
    
   <style name="ActivityDialogStyleBottom" parent="android:Theme.Dialog">
        <!--出入动画-->
        <item name="android:windowAnimationStyle">@style/AnimBottom</item>
        <!-- 边框 -->
        <item name="android:windowFrame">@null</item>
        <!-- 是否浮现在activity之上 -->
        <item name="android:windowIsFloating">true</item>
        <!-- 半透明 -->
        <item name="android:windowIsTranslucent">true</item>
        <!-- 无标题 -->
        <item name="android:windowNoTitle">true</item>
        <!-- 背景透明 -->
        <item name="android:windowBackground">@android:color/transparent</item>
        <!-- 模糊 -->
        <item name="android:backgroundDimEnabled">true</item>
    </style>
```

c.AndroidManifest配置以及使用

``` java
        <activity 
        				android:name=".fourdwallpaper.ActivityDialog" 
         				android:theme="@style/MyDialogStyleBottom"/>
      //最为重要的就是在AndroidManifest中配置theme
      
     //Acitvity以及Fragment中使用时
   startActivity(new Intent(this,ActivityDialog.class))；//activity使用
   startActivity(new Intent(getActivity(),ActivityDialog.class)); //fragment使用
```

2.DialogFragment实现

a.创建一个Fragment 继承DialogFragment

``` java
public class BottomDialogFragment extends DialogFragment {
    private View frView;
    private Window window;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        // 去掉默认的标题
        getDialog().requestWindowFeature(Window.FEATURE_NO_TITLE);
        frView = inflater.inflate(R.layout.xxxxx, null);
        return frView;
    }

    @Override
    public void onStart() {
        super.onStart();
        // 下面这些设置必须在此方法(onStart())中才有效
        window = getDialog().getWindow();
        // 如果不设置这句代码, 那么弹框就会与四边都有一定的距离
        window.setBackgroundDrawableResource(android.R.color.transparent);
        // 设置动画
        window.setWindowAnimations(R.style.bottomDialog);
        WindowManager.LayoutParams params = window.getAttributes();
        params.gravity = Gravity.BOTTOM;
        // 如果不设置宽度,那么即使你在布局中设置宽度为 match_parent 也不会起作用
        params.width = getResources().getDisplayMetrics().widthPixels;
        window.setAttributes(params);
    }
}
```

b.这里的R.style.bottomDialog 文件就是animation文件在style中的配置，然后调用的方式就跟普通的dialogFragment调用方式一致

3.PopupWindow

a.创建一个类CustomPopWindow继承PopupWindow

``` java
public class CustomPopWindow extends PopupWindow {
    private static final String TAG = "CustomPopWindow";

    private final View view;
    private View.OnClickListener itemClick;
    private Activity context;
    private TextView mTitle;
    private TextView mContent;
    private ImageView mClose;
    private AppCompatRatingBar compatRatingBar;
    private EditText mEmail;
    private EditText mSuggestion;
    private Button mSubmit;

    public CustomPopWindow(Activity context, View.OnClickListener itemClick) {
        super(context);
        LayoutInflater inflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
        view = inflater.inflate(R.layout.activity_custom_pop_window, null);
        this.itemClick = itemClick;
        this.context = context;
        initView();
        initPopWindow();
        setListener();
    }

    private void initView() {
        mTitle = view.findViewById(R.id.tv_title);
        mContent = view.findViewById(R.id.tv_content);
        mClose = view.findViewById(R.id.iv_close);
        compatRatingBar = view.findViewById(R.id.ratingBar);
        mEmail = view.findViewById(R.id.et_email);
        mSuggestion = view.findViewById(R.id.et_suggestion);
        mSubmit = view.findViewById(R.id.btn_submit);
    }

    /**
     * PopWindow 基础设置及配置
     */
    private void initPopWindow() {
        this.setContentView(view);
        // 设置弹出窗体的宽
        this.setWidth(ViewGroup.LayoutParams.MATCH_PARENT);
        // 设置弹出窗体的高
        this.setHeight(ViewGroup.LayoutParams.WRAP_CONTENT);
        // 设置弹出窗体可点击()
        this.setFocusable(false);
        this.setOutsideTouchable(false);
        //设置SelectPicPopupWindow弹出窗体动画效果
        this.setAnimationStyle(R.style.bottom_dialog_anim_style);
        // 实例化一个ColorDrawable颜色为半透明
        ColorDrawable dw = new ColorDrawable(0x00FFFFFF);
        //设置弹出窗体的背景
        this.setBackgroundDrawable(dw);
        backgroundAlpha(context, 0.5f);//0.0-1.0

    }

    /**
     * =控件监听，用于实时通过显示具体对应的UI
     */
    private void setListener() {
        compatRatingBar.setOnRatingBarChangeListener((ratingBar, rating, fromUser) -> {
            if (fromUser) {
                setRatingShowType(rating);
            }
        });
        mClose.setOnClickListener(v -> dismiss());
        mEmail.setOnClickListener(itemClick);
        mSuggestion.setOnClickListener(itemClick);
        mSubmit.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(context, "popWindow", Toast.LENGTH_SHORT).show();
            }
        });
    }

    /**
     * 通过评分具体设置对应UI的显示
     *
     * @param rating
     */
    private void setRatingShowType(float rating) {
        if (rating > 2.5f) {
            mTitle.setText(context.getResources().getString(R.string.rating_dialog_title1));
            mContent.setText(context.getResources().getString(R.string.rating_dialog_content));
            mEmail.setVisibility(View.GONE);
            mSuggestion.setVisibility(View.GONE);
            mSubmit.setVisibility(View.GONE);
            compatRatingBar.setVisibility(View.VISIBLE);
        } else {
            mTitle.setText(context.getResources().getString(R.string.rating_dialog_title2));
            mContent.setText(context.getResources().getString(R.string.rating_dialog_feedback));
            mEmail.setVisibility(View.VISIBLE);
            mSuggestion.setVisibility(View.VISIBLE);
            mSubmit.setVisibility(View.VISIBLE);
            compatRatingBar.setVisibility(View.GONE);
            this.setFocusable(true);
        }
    }

    /**
     * 设置添加屏幕的背景透明度(值越大,透明度越高)
     *
     * @param bgAlpha
     */
    public void backgroundAlpha(Activity context, float bgAlpha) {
        compatRatingBar.setRating(5.0f);
        setRatingShowType(5.0f);
        WindowManager.LayoutParams lp = context.getWindow().getAttributes();
        lp.alpha = bgAlpha;
        context.getWindow().addFlags(WindowManager.LayoutParams.FLAG_DIM_BEHIND);
        context.getWindow().setAttributes(lp);
    }
}
```

4.dialog实现

a.创建一个类继承Dialog

```java
public class RatingDialog extends Dialog {
    private static final String TAG = "RatingDialog";
    private Activity context;
    private String uri = "https://play.google.com/store/apps/details?id=com.live.parallax3d.wallpaper";
    public TextView mTitle;
    private TextView mContent;
    private ImageView mClose;
    public AppCompatRatingBar compatRatingBar;
    private EditText mEmail;
    private EditText mSuggestion;
    private Button mSubmit;
    private ImageView mHand;

    public RatingDialog(Activity context) {
        super(context);
        this.context = context;
    }

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_custom_pop_window);
        setCanceledOnTouchOutside(false);
        //setCancelable(false);
        Window window = this.getWindow();
        window.setGravity(Gravity.BOTTOM);
        window.setWindowAnimations(R.style.bottom_dialog_anim_style);
        window.setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));
        //设置dialog横向占比为全屏
        window.setLayout(WindowManager.LayoutParams.MATCH_PARENT, WindowManager.LayoutParams.WRAP_CONTENT);
        initView();
        setListener();
    }

    private void initView() {
        mTitle = findViewById(R.id.tv_title);
        mContent = findViewById(R.id.tv_content);
        findViewById(R.id.tv_star_5).setOnClickListener(v -> startWebView(context, uri));
        mClose = findViewById(R.id.iv_close);
        mHand = findViewById(R.id.iv_hand);
        compatRatingBar = findViewById(R.id.ratingBar);
        mEmail = findViewById(R.id.et_email);
        mSuggestion = findViewById(R.id.et_suggestion);
        mSubmit = findViewById(R.id.btn_submit);
    }

    /**
     * =控件监听，用于实时通过显示具体对应的UI
     */
    private void setListener() {
        compatRatingBar.setOnRatingBarChangeListener((ratingBar, rating, fromUser) -> {
            if (fromUser) {
                setRatingShowType(rating);
                if (rating > 3.0f) {
                    startWebView(context, uri);
                }
            }
        });
        mClose.setOnClickListener(v -> Dismiss());
        mSubmit.setOnClickListener(v -> {
            String email = mEmail.getText().toString();
            String suggestion = mSuggestion.getText().toString();
            if (!TextUtils.isEmpty(email) && !TextUtils.isEmpty(suggestion)) {
                FirebaseTracker.getInstance().track(MyTracker.FEED_BACK_PAGE_SUBMIT_CLICK);
                mSubmit.setEnabled(false);
                requestFeedBack(email, suggestion);
            }else {
                Toast.makeText(context, "Email and Suggestions are all need to fill in", Toast.LENGTH_SHORT).show();
            }
        });
    }

    /**
     * 跳转到指定的url
     *
     * @param context
     * @param url     指定Url
     */
    public void startWebView(Context context, String url) {
        try {
            Intent intent = new Intent();
            intent.setAction("android.intent.action.VIEW");
            Uri content_url = Uri.parse(url);
            intent.setData(content_url);
            BasePreference.getInstance().setBoolean(Constants.RATING_VALUE, true);
            Dismiss();
            context.startActivity(intent);
        } catch (ActivityNotFoundException e) {
            e.printStackTrace();
        }
    }

    /**
     * 重写关闭对话框方法，保证在每次选择评分较低后下次出现时依旧是 5 星状态
     */
    public void Dismiss() {
        compatRatingBar.setRating(5.0f);
        setRatingShowType(5.0f);
        dismiss();
    }

    @Override
    public void onBackPressed() {
        Dismiss();
    }

    /**
     * 提交反馈信息
     *
     * @param email
     * @param suggestion
     */
    private void requestFeedBack(String email, String suggestion) {
        Map<String, String> paramMap = new LinkedHashMap<>();
        paramMap.put("pkgname", getContext().getPackageName());
        paramMap.put("version", SystemUtils.getVersionName(getContext()));
        paramMap.put("vercode", String.valueOf(SystemUtils.getVersionCode(getContext())));
        paramMap.put("did", MD5Util.getStringMD5(SystemUtils.getAndroidId(getContext())));
        paramMap.put("deviceModel", android.os.Build.MODEL);
        paramMap.put("os", android.os.Build.VERSION.RELEASE);
        paramMap.put("language", SystemUtils.getCurrentLanguage());
        paramMap.put("country", SystemUtils.getCurrentCountry());
        paramMap.put("channelId", SystemUtils.getChannelId(getContext()));
        paramMap.put("resolution", String.valueOf(SystemUtils.getWindowHeight(getContext())));
        paramMap.put("cpu", android.os.Build.BOARD);
        paramMap.put("email", email);
        paramMap.put("message", suggestion);

        RetrofitNetwork.INSTANCE.getRequest().postFeedBack(paramMap).enqueue(new CommonCallback<JsonElement>() {
            @Override
            public void onResponse(JsonElement response) {
                Toast.makeText(context, "We have received your feedback , thanks !", Toast.LENGTH_SHORT).show();
                BasePreference.getInstance().setBoolean(Constants.RATING_VALUE, true);
                mSubmit.setEnabled(true);
                mEmail.setText(null);
                mSuggestion.setText(null);
                Dismiss();
            }

            @Override
            public void onFailure(Throwable t) {
                mSubmit.setEnabled(true);
                Toast.makeText(context, "feedback failed", Toast.LENGTH_SHORT).show();
            }
        });
    }

    /**
     * 通过评分具体设置对应UI的显示
     *
     * @param rating
     */
    private void setRatingShowType(float rating) {
        if (rating > 3.0f) { //Rate Us 评分页面
            mTitle.setText(context.getResources().getString(R.string.rating_dialog_title1));
            mContent.setText(context.getResources().getString(R.string.rating_dialog_content));
            mEmail.setVisibility(View.GONE);
            mSuggestion.setVisibility(View.GONE);
            mSubmit.setVisibility(View.GONE);
            compatRatingBar.setVisibility(View.VISIBLE);
            mHand.setVisibility(View.VISIBLE);
        } else { // Feedback 反馈页面
            mTitle.setText(context.getResources().getString(R.string.rating_dialog_title2));
            mContent.setText(context.getResources().getString(R.string.rating_dialog_feedback));
            mEmail.setVisibility(View.VISIBLE);
            mSuggestion.setVisibility(View.VISIBLE);
            mSubmit.setVisibility(View.VISIBLE);
            compatRatingBar.setVisibility(View.GONE);
            mHand.setVisibility(View.GONE);
        }
    }
}
```

b.使用时在Activity里面

```java
public void showDialog() {
    RatingDialog  mRatingDialog = new RatingDialog(this);
        if (!mRatingDialog.isShowing() && mRatingDialog != null) {
            mRatingDialog.show();
        }
    }
```

c.如果是Activity对应多个Fragment的情况下，直接调用在Activity里面showDialog方法就行了

三、关于上面实现App评分框的布局文件

1.layout布局文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_rating"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_gravity="bottom"
    android:background="@drawable/bg_popwindon"
    android:visibility="visible">

    <ImageView
        android:id="@+id/iv_close"
        android:layout_width="36dp"
        android:layout_height="36dp"
        android:layout_gravity="right"
        android:padding="10dp"
        android:src="@drawable/ic_close_grey" />

    <TextView
        android:id="@+id/tv_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal|top"
        android:layout_marginTop="16dp"
        android:text="@string/rating_dialog_title1"
        android:textColor="#1A1A1A"
        android:textSize="18sp"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/tv_content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal|top"
        android:layout_marginLeft="24dp"
        android:layout_marginTop="48dp"
        android:layout_marginRight="24dp"
        android:gravity="center"
        android:text="@string/rating_dialog_content"
        android:textColor="#313131"
        android:textSize="17sp" />

    <androidx.appcompat.widget.AppCompatRatingBar
        android:id="@+id/ratingBar"
        style="@style/RatingStyle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center_horizontal|top"
        android:layout_marginTop="110dp"
        android:layout_marginBottom="50dp"
        android:isIndicator="false"
        android:rating="5"
        android:stepSize="1"
        android:focusable="true"
        android:visibility="visible" />

    <TextView
        android:id="@+id/tv_star_5"
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:layout_marginTop="110dp"
        android:layout_marginBottom="50dp"
        android:layout_marginLeft="96dp"
        android:layout_gravity="center_horizontal"/>

    <ImageView
        android:id="@+id/iv_hand"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/ic_rading_hand"
        android:layout_marginTop="120dp"
        android:layout_marginBottom="40dp"
        android:layout_gravity="center_horizontal"
        android:layout_marginLeft="130dp"/>

    <EditText
        android:id="@+id/et_email"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:layout_marginLeft="24dp"
        android:layout_marginTop="120dp"
        android:layout_marginRight="24dp"
        android:layout_marginBottom="150dp"
        android:background="@drawable/bg_rating_feedback"
        android:gravity="center_vertical"
        android:hint="Your Email..."
        android:maxLines="1"
        android:paddingLeft="16dp"
        android:paddingRight="16dp"
        android:textSize="16sp"
        android:visibility="gone" />

    <EditText
        android:id="@+id/et_suggestion"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:layout_marginLeft="24dp"
        android:layout_marginTop="175dp"
        android:layout_marginRight="24dp"
        android:layout_marginBottom="70dp"
        android:background="@drawable/bg_rating_feedback"
        android:gravity="left"
        android:hint="Suggestion..."
        android:paddingLeft="16dp"
        android:paddingTop="5dp"
        android:paddingRight="16dp"
        android:textSize="16sp"
        android:visibility="gone" />

    <Button
        android:id="@+id/btn_submit"
        android:layout_width="100dp"
        android:layout_height="40dp"
        android:layout_gravity="right"
        android:layout_marginTop="255dp"
        android:layout_marginRight="24dp"
        android:layout_marginBottom="15dp"
        android:background="@drawable/bg_feedback_btn"
        android:text="SUBMIT"
        android:textColor="#FFFFFF"
        android:textSize="18sp"
        android:visibility="gone" />

</FrameLayout>
```

2.再者就是style中的style配置，主要是对Rating控件的配置

```xml
 <!--设置评分控件属性-->
    <style name="RatingStyle" parent="Widget.AppCompat.RatingBar">
        <item name="android:progressDrawable">@drawable/selector_rating_bar</item>
        <item name="android:minHeight">30dp</item>
        <item name="android:maxHeight">30dp</item>
    </style>
```

3.文件selector_rating_bar

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!--  ic_start_normal_t  是未选中下的星星图标
			 ic_star_press_t  是选中情况下的星星图标-->
    <item android:id="@android:id/background"
        android:drawable="@drawable/ic_start_normal_t"/> 

    <item android:id="@android:id/secondaryProgress"
        android:drawable="@drawable/ic_start_normal_t"/>
    
    <item android:id="@android:id/progress"
        android:drawable="@drawable/ic_star_press_t"/>

</layer-list>
```

四、总结

​		起初我是使用的PopupWindow ,基本效果都实现了，效果也可以，但是有一个地方会与需求冲突，就是在保证弹窗show后点击窗口外部不会使弹窗关闭，`this.setFocusable(false)`;同时这个方法会使弹窗失去焦点，如果你弹窗里面存在EditText是无法弹出输入法，必须将setFocusable设置成true,因为需要这2种同时存在，PopipWindow并不能满足我的需求，同时全局Activity也不能满足我的需求，基于项目开发是基于一个Acticity上实现多个Fragment,最后选择使用Dialog方式去实现需求，Dialog中设置`setCanceledOnTouchOutside(false);`并不会使弹窗失去焦点，同时使用EditText时也会按需求弹出输入法；FragmentDialog我并没有去尝试，但是有坑的地方就在焦点处理方面，如果仅仅是需要做一个底部类似与分享弹窗的话，上述几种方案都能实现具体需求，在没有焦点冲突时，PopuoWindow使用还是比Dialog处理起来更方便一点。