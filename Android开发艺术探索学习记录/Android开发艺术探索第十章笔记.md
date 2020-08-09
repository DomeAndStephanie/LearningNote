### Android的消息机制

---

1. #### Android的消息机制概述

   Android的消息机制主要指**Handler**的运行机制以及**Handler**所附带的**MessageQueue**和**Looper**的工作过程，Handler的主要作用是将一个任务切换到某个指定的线程中去执行。Handler创建时会采用当前线程的Looper来构建内部的消息循环系统，如果没有Looper会报错。线程中默认是没有Looper的，主线程（UI线程，即ActivityThread）被创建时会初始化Looper，所以主线程默认可以使用Handler

   ![图10-1Handler的工作过程](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图10-1Handler的工作过程.png)

2. #### Android的消息机制分析

   - ThreadLocal的工作原理

     ThreadLocal:是一个线程内部的数据存储类，可以通过它在指定的线程存储和获取数据（其他线程无法获取）

     ```java
     //定义
     public class ThreadLocal<T>
     
     //set方法
     public void set(T value) {
             Thread t = Thread.currentThread();
             ThreadLocalMap map = getMap(t);
             if (map != null)
                 map.set(this, value);
             else
                 createMap(t, value);
       }
       
     //get方法
         public T get() {
             Thread t = Thread.currentThread();
             ThreadLocalMap map = getMap(t);
             if (map != null) {
                 ThreadLocalMap.Entry e = map.getEntry(this);
                 if (e != null) {
                     @SuppressWarnings("unchecked")
                     T result = (T)e.value;
                     return result;
                 }
             }
             return setInitialValue();
         }
         
     ```
     
   - 消息队列的工作原理

     MessageQueue主要包含两个操作：**插入**和**读取**，对应*enqueueMessage*和*next*，MessageQueue是通过一个单链表的数据结构来维护消息列表的，不是队列。

     > enqueueMessage方法的主要操作是单链表的插入操作
     >
     > next方法的主要操作是构建一个无限循环，有消息时返回这条消息并移除；无消息时则进行阻塞。

   - Looper的工作原理

     Looper是消息循环，会不停的查看MessageQueue中是否有新消息，有则立即处理；无则进行阻塞。

     重要方法：

     `prepare` : 为当前线程创建一个Looper

     `prepareMainLooper` : ActivityThread创建Looper使用

     `getMainLooper` : 获取主线程的Looper

     `quit` : 直接退出Looper

     `quitSafely` : 设定退出标记，在把消息队列中已有消息处理完后安全退出

     `loop` : 调用MessageQueue的next方法来获取新消息

   - Handler的工作原理

     **Handler**的工作主要包含消息的发送和接收过程。

     发送消息的过程为向消息队列中插入一条消息，随后**MessageQueue**返回消息给**Looper**，**Looper**在交由**Handler**处理，即调用**Handler**的*dispatchMessage*方法

     ![image-20200803223746136](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图10-2Handler消息处理流程.png)

3. #### 主线程的消息循环

   主线程即ActivityThread,其入口方法为main，在main方法中通过Looper.prepareMainLooper来创建主线程的Looper以及MessageQueue，并通过Looper.loop方法来开启主线程的消息循环

   ```java
       public static void main(String[] args) {
           Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
   
           // Install selective syscall interception
           AndroidOs.install();
   
           // CloseGuard defaults to true and can be quite spammy.  We
           // disable it here, but selectively enable it later (via
           // StrictMode) on debug builds, but using DropBox, not logs.
           CloseGuard.setEnabled(false);
   
           Environment.initForCurrentUser();
   
           // Make sure TrustedCertificateStore looks in the right place for CA certificates
           final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
           TrustedCertificateStore.setDefaultUserDirectory(configDir);
   
           Process.setArgV0("<pre-initialized>");
   
           Looper.prepareMainLooper();
   
           // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
           // It will be in the format "seq=114"
           long startSeq = 0;
           if (args != null) {
               for (int i = args.length - 1; i >= 0; --i) {
                   if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                       startSeq = Long.parseLong(
                               args[i].substring(PROC_START_SEQ_IDENT.length()));
                   }
               }
           }
           ActivityThread thread = new ActivityThread();
           thread.attach(false, startSeq);
   
           if (sMainThreadHandler == null) {
               sMainThreadHandler = thread.getHandler();
           }
   
           if (false) {
               Looper.myLooper().setMessageLogging(new
                       LogPrinter(Log.DEBUG, "ActivityThread"));
           }
   
           // End of event ActivityThreadMain.
           Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
           Looper.loop();
   
           throw new RuntimeException("Main thread loop unexpectedly exited");
       }
   ```

   ActivityThread通过ActivityThread.H来和消息队列进行交互。具体流程为：ActivityThread通过ApplicationThread和ActivityManagerService进行进程间通信，AMS以进程间通信的方式完成ActivityTread的请求后会回调ApplicationThread中的Binder方法，然后ApplicationThread会向H发送消息，H收到消息后将ApplicationThread中的逻辑切换到ActivityThread中去执行，即切换到主线程执行。
