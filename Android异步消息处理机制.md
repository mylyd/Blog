### 笔记主目录

#### 异步消息据机制

我们先说下什么是Android消息处理机制？消息处理机制本质：一个线程开启循环模式持续监听并依次处理其他线程给它发的消息。简单的说：一个线程开启一个无限循环模式，不断遍历自己的消息列表，如果有消息就挨个拿出来做处理，如果列表没消息，自己就堵塞（相当于wait，让出cpu资源给其他线程），其他线程如果想让该线程做什么事，就往该线程的消息队列插入消息，该线程会不断从队列里拿出消息做处理。

-   **Looper **

    我们知道一个线程是一段可执行的代码，当可执行代码执行完成后，线程生命周期便会终止，线程就会退出，那么做为App的主线程，如果代码段执行完了会怎样？，那么就会出现App启动后执行一段代码后就自动退出了，这是很不合理的。所以为了防止代码段被执行完，只能在代码中插入一个死循环，那么代码就不会被执行完，然后自动退出，怎么在在代码中插入一个死循环呢？那么Looper出现了，在主线程中调用`Looper.prepare()...Looper.loop()`就会变当前线程变成Looper线程（可以先简单理解：无限循环不退出的线程），Looper.loop()方法里面有一段死循环的代码，所以主线程会进入`while(true){...}`的代码段跳不出来，但是主线程也不能什么都不做吧？其实所有做的事情都在while(true){...}里面做了，主线程会在死循环中不断等其他线程给它发消息（消息包括：Activity启动，生命周期，更新UI，控件事件等），一有消息就根据消息做相应的处理，Looper的另外一部分工作就是在循环代码中会不断从消息队列挨个拿出消息给主线程处理。



-   **MessageQueue **

    MessageQueue 存在的原因很简单，就是同一线程在同一时间只能处理一个消息，同一线程代码执行是不具有并发性，所以需要队列来保存消息和安排每个消息的处理顺序。多个其他线程往UI线程发送消息，UI线程必须把这些消息保持到一个列表（它同一时间不能处理那么多任务),然后挨个拿出来处理，这种设计很简单，我们平时写代码其实也经常这么做。每一个Looper线程都会维护这样一个队列，而且仅此一个，这个队列的消息只能由该线程处理。



- **Handler **

    简单说Handler用于同一个进程的线程间通信。Looper让主线程无限循环地从自己的MessageQueue拿出消息处理，既然这样我们就知道**处理消息肯定是在主线程中处理的**，那么怎样在其他的线程往主线程的队列里放入消息呢？其实很简单，我们知道在同一进程中线程和线程之间资源是共享的，也就是对于任何变量在任何线程都是可以访问和修改的，只要考虑并发性做好同步就行了，那么只要拿到MessageQueue 的实例，就可以往主线程的MessageQueue放入消息，主线程在轮询的时候就会**在主线程**处理这个消息。那么怎么拿到主线程 MessageQueue的实例，是可以拿到的（获取方式`mLooper = Looper.myLooper();mQueue = mLooper.mQueue;`），Handler 在sendMessage的时候就通过这个引用往消息队列里插入新消息。Handler 的另外一个作用，就是能统一处理消息的回调。这样一个Handler发出消息又确保消息处理也是自己来做，这样的设计非常的赞。具体做法就是在队列里面的Message持有Handler的引用（哪个handler 把它放到队列里，message就持有了这个handler的引用），然后等到主线程轮询到这个message的时候，就来回调我们经常重写的Handler的handleMessage(Message msg)方法。




-   **Message **

    Message 很简单了，你想让主线程做什么事，总要告诉它吧，总要传递点数据给它吧，Message就是这个载体。

    ​


![](image/Android异步消息机制图解.png)



**更新UI的几种方法**

1. Handler 的 `handleMessage()`  方法
2. Handler 的 `post()` 方法
3. View 的 `post()` 方法
4. Activity 的 `runOnUiThread()` 方法



* Handler 的 `handleMessage()`  方法

  ```java
  //主线程中 new Handler 实例
  Handler handler = new Handler(){
      public void handleMessage(Message msg){
          super.handleMessage(msg);
          
          //判断消息发送者
          switch(msg.what){
              case 1 :
              	// 更新UI操作
                  
              back;
          }
      }
  }

  //子线程中 执行耗时操作
  new Thread(new Runnable() {
      @Override
      public void run() {
          
          //返回结果给主线程更新UI
  		Message message = new Message();  
           message.what = 1;  
           Bundle bundle = new Bundle();  
           bundle.putString("aaa", "bbb");  
           message.setData(bundle);  
           handler.sendMessage(message);  
      }
  }).start();

  ```

  ​

* Handler 的 `post()` 方法

  ```java
  //主线程中 new Handler 实例
  Handler handler = new Handler();

  //开启子线程
  new Thread(new Runnable(){
      
      public void run(){
          
          //耗时操作
          
          handler,post(new Runnable(){
              
              public void run(){
                  
                  //在这里进行 UI 操作  
              }
          })
      }
  }).start();
  ```

  ​

