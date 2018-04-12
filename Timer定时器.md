

### [笔记主目录](https://mylyd.github.io/myhome.html)

#### 定义定时任务

定时器会开启子线程,  **无法更新UI**

```java
    Timer timer = new Timer();
    TimerTask task = new TimerTask() {
        @Override
        public void run() {
           
            //要定时执行的代码
            
           //使用handler更新UI
            
        }
    };


```

[Android异步消息处理机制](Android异步消息处理机制.md)



**开启定时器** [^1]  `timer.schedule(task,0,2000);`

-   开启定时器需要三个参数

1.  就是上面所写的你要做的事情



2.  开启延时，0秒就是立即执行定时器
3.  执行间隔 (周期)，如果不写则默认只执行一次




**注销定时器**  [^2]    `timer.cancel();`

[^1]: 一般在 `onStart()`里注销
[^2]: 一般在 `onStop()`里注销