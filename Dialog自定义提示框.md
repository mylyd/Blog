### [笔记主目录](https://mylyd.github.io/myhome.html)

#### Dialog自定义提示框

## 代码（Java）

```java
class DialogThis{
        private Button dialog_btn;
        private TextView dialog_text;
        private View view;

        private void initDialog(){
            view=View.inflate(MainActivity.this,R.layout.style_dialog,null);
            dialog_btn= (Button) view.findViewById(R.id.btn_Determine);
            dialog_text= (TextView) view.findViewById(R.id.dialog_text);
        }
        public void ondialog(){
            AlertDialog.Builder odialog=new AlertDialog.Builder(MainActivity.this);
            initDialog();
            odialog.setView(view);
            dialog_text.setText("网络已连接");
            odialog.setCancelable(false);
            final AlertDialog onDialog=odialog.create();
            dialog_btn.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                   onDialog.cancel();
                }
            });
            onDialog.show();
        }
        public void undialog(){
            AlertDialog.Builder udialog=new AlertDialog.Builder(MainActivity.this);
            initDialog();
            udialog.setView(view);
            dialog_text.setText("网络已断开");
            udialog.setCancelable(false);
            final AlertDialog unDialog=udialog.create();
            dialog_btn.setOnClickListener(new View.OnClickListener() {
               @Override
               public void onClick(View v) {
                   unDialog.cancel();
               }
           });
           unDialog.show();
        }
    }
```

## 效果图

![dialog](C:\Users\13466\Desktop\image\dialog.png)

## xml (layout : style_dialog)

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView
        android:id="@+id/title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="left"
        android:padding="10dp"
        android:layout_marginLeft="10dp"
        android:text="提 示"
        android:textColor="#00ffff"
        android:textSize="24sp" />
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:background="#039b9b"/>
    <TextView
        android:id="@+id/dialog_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="20dp"
        android:layout_marginLeft="10dp"
        android:textColor="#000000"
        android:textSize="22sp" />
    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:background="#dfdede"/>

    <LinearLayout
        android:layout_width="match_parent"
        android:gravity="center"
        android:layout_height="60dp">
        <Button
            android:id="@+id/btn_Determine"
            android:layout_width="match_parent"
            android:layout_gravity="center"
            android:layout_height="match_parent"
            android:background="#00000000"
            android:textSize="24dp"
            android:text="确 定" />
    </LinearLayout>
</LinearLayout>
```

​	< 或者 >

```java
//定义一个初始化工具
public class DialogFactory {
    static Dialog dialog = null;
    public interface OnListener {
        void ondialog();
    }
    
    public static void showDialog(Context context, String title, String msg,
                                  final OnListener oAfter) {
        AlertDialog.Builder builder = new AlertDialog.Builder(context, AlertDialog.THEME_HOLO_LIGHT);
        //AlertDialog.THEME_HOLO_LIGHT***这是样式，
        /*1.THEME_TRADITIONAL
          2.THEME_HOLO_DARK
          3.THEME_HOLO_LIGHT
          4.THEME_DEVICE_DEFAULT_DARK
        */
        builder.setTitle(title);
        builder.setMessage(msg);
        builder.setNegativeButton("取消", null);
        builder.setPositiveButton("确定", new OnClickListener() {

            @Override
            public void onClick(DialogInterface dialog, int which) {
                oAfter.onAfter();
                //dialog.dismiss();
            }
        });
        dialog = builder.create();
        if (!dialog.isShowing()) {
            dialog.show();
        }
    }
}



//调用
DialogFactory.showDialog(context,titlt,str,new DialogFactory.OnListener() {
                    @Override
                    public void onAfter() {
                      
                    }
                });
```

