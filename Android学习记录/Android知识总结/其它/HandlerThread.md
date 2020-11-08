[TOC]

#### HandlerThread

---

##### 一. 概述

HandlerThread继承了`Thread`,本质上还是一个`Thread`,因此启动时仍然需要使用==start==进行启动，不同之处在于该类中带有一个`Looper`用来创建`Handler`,HandlerThread通过搭配该Handler分发处理消息执行任务。

##### 二. 使用方法

- 创建HandlerThread实例对象并启动

  ```java
  HandlerThread handlerThread = new HandlerThread("MainActivity");
  handlerThread.start();
  ```

- 基于HandlerThread创建Handler

  ```java
  Handler handler = new Handler(handlerThread.getLooper()){
              @Override
              public void handleMessage(@NonNull Message msg) {
                  super.handleMessage(msg);
                  
              }
  };
  ```

- 使用Handler分发处理消息

- 退出Looper死循环

  ```
  handlerThread.quit();
  ```

##### 三. 源码阅读

```java
public class HandlerThread extends Thread {
    //线程优先级
    int mPriority;
    //线程Id
    int mTid = -1;
    //当前线程持有的Looper，用来创建Handler
    Looper mLooper;
    //用于线程间通信
    private @Nullable Handler mHandler;

    //构造方法，可以设置线程名和优先级
    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }
    
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }
    
    //如果要在Looper循环启动前进行操作，需要覆写这个回调
    protected void onLooperPrepared() {
    }

    @Override
    public void run() {
        //记录当前的线程ID
        mTid = Process.myTid();
        //触发当前线程创建Looper对象
        Looper.prepare();
        synchronized (this) {
            //获取当前线程绑定的Looper,未绑定则为null
            mLooper = Looper.myLooper();
            //唤醒getLooper()中处于等待的线程
            notifyAll();
        }
        //设置线程优先级
        Process.setThreadPriority(mPriority);
        //在消息循环之前调用回调
        onLooperPrepared();
        //开启消息循环
        Looper.loop();
        mTid = -1;
    }
    
    //获取当前线程的Looper对象
    public Looper getLooper() {
        //如果当前线程未运行，直接返回null
        if (!isAlive()) {
            return null;
        }
        //如果当前线程已经启动了，但是Looper还没有创建，调用wait方法释放锁等待创建Looper
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

    //返回与当前线程关联的Handler
    @NonNull
    public Handler getThreadHandler() {
        if (mHandler == null) {
            mHandler = new Handler(getLooper());
        }
        return mHandler;
    }
    
    //清空消息队列中的所有消息，调用该方法可能会让一些消息丢失，遇到该情况可使用quitSafely方法
    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }

    //清空消息队列中所有非延时消息
    public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

    //返回当前线程ID
    public int getThreadId() {
        return mTid;
    }
}
```

