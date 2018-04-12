### 笔记主目录

#### Android接口回调

#####   首先是A类

```java
public class A{
    
    //1个接口变量
    private Callback mCallback;
   

    //1个接口
    public interface Callback{
        void doSomething();
    }
   

    //1个给接口变量赋值的方法
    public void setCallback(Callback callback){
        mCallback = callback;
    }
   

    //1个使用接口变量的地方
    public void onExecute(){
        
        //会调用B类的doSomething()的具体实现
        mCallback.doSomething();
       
    }
    
}
```
#####    然后是B类 实现接口

```java
public class B implements A.Callback{   

    //A类的实例的引用
    private A mAInstance;
    

    //B类实现了A类的接口
    public void doSomething(){
        Log.d("TAG","will do something");
    }
    
    //B类将自己（实际上是接口的实现）传给A类实例的接口变量
    mAInstance.setCallback(this);       
}

```

#####   或者是 使用匿名内部类

```java
public class B{
    
    //A类的实例的引用
    private A mAInstance;
    
    //匿名内部类实现A类Callback()接口
    mAInstance.setCallback(new A.Callback (){
        //实现接口的抽象方法
        void doSomething(){
            Log.d("TAG","will do something");
        }
    });
}

```

#####   复习

-   接口中的所有方法均为抽象方法
-   实现接口的类必须实现接口中的全部方法

#####  总结一下就是以下的几个点：

-   A类中有一个接口变量和接口。
-   B类实现A类的接口（这个接口就是所谓的回调接口）。
-   A类的接口变量会（通过过某种方法）获得靠B类实现的接口。
-   A类在一个适当的时机“使用”这个接口变量，即调用接口中的函数（这个函数就是所谓的回调函数）。
-   回调函数不应该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。 

##### 举例

 A类中有一个 __*工作接口*__  , 里面有一个 __*工作方法*__  ,  B类是 __员工__  类 , 实现 __*工作接口*__  , 并实现接口里面的 __*工作方法*__ 为 写代码 , 然后 A 类在早上九点钟时(触发条件 : 使用接口变量的地方) , 调用 __*工作方法*__  , B类就会执行 __*工作方法*__  开始写代码



##### 实际生活中的例子

 去公司面试 

在面试之前会先写一份简历 , 然后向面试公司投递简历 , 面试公司收到简历后 , 会一段时间后给予面试回复(可能会给予不予面试回复) , 收到面试回复就就前去公司开始自己的面试  

在这个例子中 

1.  *写简历*  即 声明 A类的实例的引用 `private A mAInstance;`

2.  *投递简历*  即 B类将 自己[^1]传给A类实例的接口变量`mAInstance.setCallback(this);  `

3.  *面试公司收到简历*  即 给接口变量赋值

  ```java
    public void setCallback(Callback callback){
        mCallback = callback;
    }
  ```

4.  *一段时间后给予回复*  即 使用接口变量的地方

  ```java
  public void onExecute(){
     //会调用B类的doSomething()的具体实现
      mCallback.doSomething();
  }
  ```

5.  *开始自己的面试*  即  执行 具体的接口方法

  ```java
  public void doSomething(){
      Log.d("TAG","will do something");
  }
  ```


#####注释

[^1]: 实际是接口的实现 
