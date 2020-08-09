### Android中的线程和线程池

---

在操作系统中，线程是调度的最小单元，同时线程不能无限制的产生，线程的创建和销毁都需要开销，通过线程池可以避免因为频繁创建和销毁线程而导致的开销。

###### 一. 主线程和子线程

- 主线程:指**进程**所拥有的线程,主要处理和界面相关的事情,也称`UI线程`.

  > 为了避免界面卡顿,主线程中不能执行耗时的任务,会引起ANR

- 子线程:也称`工作线程`,除主线程以外的线程都是子线程.主要执行耗时任务，如网络请求，I/O操作等

###### 二. Android中的线程形态

- AsyncTask

  内部封装了`Thread`和`Handler`,在线程池中执行后台任务,把执行进度和结果返回给主线程.

  > AsyncTask不适合执行特别耗时的后台任务

  ```java
  public abstract class AsyncTask<Params,Progress,Result>{
      //AsyncTask参数
      //Params:表示参数的类型
      //Progress:后台任务的执行进度的类型
      //Result:任务的返回结果的类型
      /**
      * 在主线程中执行,异步任务执行{#doInBackground}之前会被调用
      **/
      @MainThread
      protected void onPreExecute() {
      }
      /**
      * 用于执行异步任务,在此方法中可以通过publishProgress方法更新进度
      **/
      @WorkerThread
      protected abstract Result doInBackground(Params... params);
      /**
      * 在主线程中执行,用于更新任务进度
      **/
      @MainThread
      protected void onProgressUpdate(Progress... values) {
      }
      /**
      * 在主线程中执行,在异步任务执行之后调用,用于返回结果
      **/
      @MainThread
      protected void onPostExecute(Result result) {
      }
      /**
      * 执行在主线程,异步任务取消时调用,默认实现中直接调用了onCancelled()方法,所以
      * 忽略了结果值,注意在覆写时不要使用super.onCancelled(result)
      **/
      @MainThread
      protected void onCancelled(Result result) {
          onCancelled();
      }   
  }
  ```

  AsyncTask使用限制：

  1. AsyncTask的类必须在主线程加载
  2. AsyncTask的对象必须在主线程创建
  3. execute方法必须在UI线程调用
  4. 不要在程序中直接调用onPreExecute, onPostExecute, doInBackground, onProgressUpdate
  5. 一个AsyncTask对象只能执行一次，即只能调用一次execute方法

  工作原理:

  ```mermaid
  graph TB
  A["excute(Params... params)"] --> B["executeOnExecutor(sDefaultExecutor,param)"]
  B --> C["SERIAL_EXECUTOR#execute(Runnable r)"]
  C --> D["SERIAL_EXECUTOR#scheduleNext()"]
  D --> E["THREAD_POOL_EXECUTOR#execute(Runnable r)"]
  ```

  > AsyncTask中有两个线程池(SERIAL_EXECUTOR,THREAD_POOL_EXECUTOR)和一个线程Handler(InternalHandler)
  >
  > SERIAL_EXECUTOR:内部实现的`SerialExecutor`,用于任务的排队,接收到传递过来的**FutureTask**对象后将其插入到任务队列中,如果此时没有任务在执行则调用*scheduleNext*方法执行下一个任务.可以看出AsyncTask是**串行执行**的
  >
  > THREAD_POOL_EXECUTOR:用于执行任务
  >
  > InternalHandler:用于子线程切换到主线程

- HandlerThread

  HandlerThread继承自Thread,其内部构建了一个消息处理循环

  - 用法

    ```java
    //创建HandlerThread实例
    HandlerThread mMsgThread = new HandlerThread("msg-handler");
    //启动
    mMsgThread.start();
    //关联Handler处理消息
    Handler mMsgHandler = new Handler(mMsgThread.getLooper()){
        @Override
        public void handleMessage(Message msg){
            //消息处理
        }
    }
    ```

- IntentService

  IntentService是继承了*Service*的一个**抽象类**.可用于执行后台耗时的任务,**当任务执行完成后自动停止**.

  工作原理:

  ```java
  //onCreate
  //创建一个HandlerThread,并通过其再创建mServiceHandler
  @Override
  public void onCreate() {
      super.onCreate();
      HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
      thread.start();
  
      mServiceLooper = thread.getLooper();
      mServiceHandler = new ServiceHandler(mServiceLooper);
  }
  //onStart中通过mServiceHandler发送消息,然后在ServiceHandler中调用onHandleIntent
  //方法处理消息,并随后调用Service#stopSelt(startId)停止服务
  private final class ServiceHandler extends Handler {
      public ServiceHandler(Looper looper) {
          super(looper);
      }
  
      @Override
      public void handleMessage(Message msg) {
          onHandleIntent((Intent)msg.obj);
          stopSelf(msg.arg1);
      }
  }
  ```

  > - Service中的`stopSelf()`和`stopSelf(int startId)`都能停止服务,但`stopSelf()`会立刻停止服务,而`stopSelf(int startId)`会等待所有消息都处理完成后停止服务.
  > - `onHandleIntent`方法需要在子类中实现,根据Intent参数区分不同的任务并执行.
  > - `onHandleIntent`中的任务是排队执行的,执行顺序为发起请求的顺序.

###### 三. Android中的线程池

- 线程池的优点

  1. 线程复用,避免线程重复创建和销毁带来的开销
  2. 有效控制线程池的最大并发数,避免大量的线程之间因资源抢占而阻塞
  3. 统一管理线程

- ThreadPoolExecutor

  ```java
  //构造方法
  public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler) 
  ```

  1. 构造参数

     - corePoolSize:线程池的核心线程数,默认会一直存活.

     - maximumPoolSize:线程池能容纳的**最大线程数**,达到这个数值后,后续任务会被**阻塞**

     - keepAliveTime:**非核心线程**闲置时的超时时长,超过这个时间后会被回收.

       > 如果设置`allowCoreThreadTimeOut`为**True**,可作用于核心线程

     - unit:`keepAliveTime`参数时间单位

     - workQueue:线程池中的任务队列,通过线程池的`execute`方法提交的*Runnable*对象会存储在该参数中

     - threadFactory:线程工厂,为线程池提供创建新线程的功能

     - handler:当线程池无法执行新任务时,`ThreadPoolExecutor`会调用该handler的**rejectedExecution**方法,默认直接抛出`RejectedExecutionException`

  2. 执行规则

     - 如果线程池中的线程数未达到核心线程数,则直接启动核心线程执行任务
     - 如果线程数量大于或等于核心线程数,任务会被插入到任务队列中排队等待
     - 如果线程数量大于或等于核心线程数且任务队列也已满,此时如果线程数量未达到线程能容纳的最大值,则立刻启动一个非核心线程执行任务;如果达到了最大值则拒绝执行任务调用`rejectedExecution`方法

  3. 线程池的分类

     - FixedThreadPool

       ```java
       public static ExecutorService newFixedThreadPool(int nThreads) {
           return new ThreadPoolExecutor(nThreads, nThreads,
                                         0L, TimeUnit.MILLISECONDS,
                                         new LinkedBlockingQueue<Runnable>
                                         ());
       }
       ```

       通过`Executors`的`newFixedThreadPool`方法创建.一种线程数量固定的线程池,只有核心线程,没有超时机制和任务队列大小限制.

     - CachedThreadPool

       ```java
       public static ExecutorService newCachedThreadPool() {
           return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                         60L, TimeUnit.SECONDS,
                                         new SynchronousQueue<Runnable>());
       }
       ```

       通过`Executors`的`newCachedThreadPool`方法来创建.一种线程数量不定的线程池,只有非核心线程,且最大线程数为*Interger.MAX_VALUE*.线程池中的空闲线程都有超时机制,超时时间为60s

     - ScheduledThreadPool

       ```java
       public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
           return new ScheduledThreadPoolExecutor(corePoolSize);
       }
       public ScheduledThreadPoolExecutor(int corePoolSize) {
           super(corePoolSize, Integer.MAX_VALUE,
                 DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
                 new DelayedWorkQueue());
       }
       ```

       通过`Executors`的`newSingleThreadScheduledExecutor`创建.核心线程数固定,非核心线程数最大为*Interger.MAX_VALUE*,非核心线程闲置时会被回收.用于执行定时任务和具有固定周期的重复任务.

     - SingleThreadExecutor

       ```java
       public static ExecutorService newSingleThreadExecutor() {
           return new FinalizableDelegatedExecutorService
               (new ThreadPoolExecutor(1, 1,
                                       0L, TimeUnit.MILLISECONDS,
                                       new LinkedBlockingQueue<Runnable>()));
       }
       ```

       通过`Executors`的`newSingleThreadExecutor`方法创建.线程池中只有一个核心线程,确保所有的任务都在同一个线程中按顺序执行.