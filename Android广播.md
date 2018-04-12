### [笔记主目录](https://mylyd.github.io/myhome.html)

#### 一、概述

## 1.广播的分类

### (1)按照发送的方式分类

> - 标准广播
>   是一种异步的方式来进行传播的，广播发出去之后，所有的广播接收者**几乎是同一时间**收到消息的。他们之间没有先后顺序可言，而且这种广播是没法被截断的。
> - 有序广播
>   是一种同步执行的广播，在广播发出去之后，**同一时刻只有一个广播接收器可以收到消息**。当广播中的逻辑执行完成后，广播才会继续传播。

### (2)按照注册的方式分类

> - 动态注册广播
>   顾名思义，就是在代码中注册的。
> - 静态注册广播
>   动态注册要求程序必须在运行时才能进行，有一定的局限性，如果我们需要在程序还没启动的时候就可以接收到注册的广播，就需要静态注册了。主要是在AndroidManifest中进行注册。

### (3)按照定义的方式分类

> - 系统广播
>   Android系统中内置了多个系统广播，每个系统广播都具有特定的intent-filter，其中主要包括具体的action，系统广播发出后，将被相应的BroadcastReceiver接收。系统广播在系统内部当特定事件发生时，由系统自动发出。
> - 自定义广播
>   由应用程序开发者自己定义的广播

------

## 2.动态注册广播的实现

一段比较典型的实现代码为：

### (1)实现一个广播接收器

```java
public class MyBroadcastReceiver extends BroadcastReceiver {
    
	//接收广播
    @Override
    public void onReceive(Context context, Intent intent) {
       
        Toast.makeText(context, "received in MyBroadcastReceiver", Toast.LENGTH_SHORT).show();
       
    }
}
```

主要就是继承一个BroadcastReceiver，实现onReceive方法，在其中实现自己的业务逻辑就可以了。

### (2)注册广播

```java
public class MainActivity extends AppCompatActivity {

    private IntentFilter intentFilter;
    private MyBroadcastReceiver myBroadcastReceiver;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        //注册广播
        intentFilter = new IntentFilter();
        intentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
        myBroadcastReceiver = new MyBroadcastReceiver();
        registerReceiver(myBroadcastReceiver, intentFilter);
        
        //发送广播
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent("android.net.conn.CONNECTIVITY_CHANGE");
                sendBroadcast(intent); 
            }
        });
    }
	
    //注销广播
    @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(myBroadcastReceiver);
    }
}
```

这样MyBroadcastReceiver就可以收到相应的广播消息了。

------

## 3.静态注册广播的实现

还是用上面的按个MyBroadcastReceiver，只不过这次采用静态注册的方式
在manifest文件中增加如下的代码：

```xml
        <receiver
            android:name=".MyBroadcastReceiver"
            android:enabled="true"
            android:exported="true">
            <intent-filter>
                <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
            </intent-filter>
        </receiver>
```