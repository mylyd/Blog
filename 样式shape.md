# 边款与样式

```xml
<shape xmlns:android="http://schemas.android.com/apk/res/android"
       android:shape="[rectangle（矩形）|oval(圆形)|line(线性形状)|ring（环形）]">
    
    <corners android:radius="_dp"/>  //弧形按钮边角半径；
    <solid android:color="————"/>  //设置背景色         
</shape>
```

# 边框

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android" android:shape="rectangle" >
    <stroke android:width="1dp" 			//线宽
            android:color="#000000"/>

           <!-- android:dashGap="3dp"		//间隙
            android:dashWidth="6dp"//虚线-->	
</shape>
```







