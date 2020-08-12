### Android开发艺术探索 第二章笔记

#### 第二章  IPC机制

1. ##### Android IPC 简介

   IPC：Inter-Process Communication的缩写，意思为进程间通信或者跨进程通信，指两个进程之间进行数据交换的过程。

   线程：CPU调度的最小单元，是一种有限的系统资源。

   进程：一般指一个执行单元，在PC和移动设备上指一个程序或者应用。一个进程可以包含多个线程。

2. ##### Android中的多进程模式

   1. 开启多进程模式

      在Android中使用多进程只有一个方法，那就是给**四大组件**在**AndroidManifest**中指定*android:process*属性。

      > 进程名以“：”开头的进程属于当前应用的私有进程，其他应用的组件不可以和它在同一进程中，而进程名不以“：”开头的进程属于全局进程，其他应用可以通过ShareUID方式和它在同一进程中。Android系统会为每个应用分配一个唯一的UID，具有相同UID的应用才能共享数据。(两个应用通过**ShareUID**在同一个进程张，不但**ShareUID**要相同，签名也必须相同才可以互相访问数据)

   2. 多进程模式运行机制

      多进程造成的问题：

      1. 静态成员和单例模式完全失效。
      2. 线程同步机制完全失效。
      3. **SharedPreferences**的可靠性下降。
      4. **Application**会多次创建。

      > 在多进程模式中，不同进程的组件会拥有独立的虚拟机，Application以及内存空间。

3. ##### IPC基础概念介绍

   1. Serializable接口：Java所提供的一个序列化接口，是一个空接口，为对象提供序列化和反序列化操作。

      实现只要实现该接口，并且在类的声明中指定下面的标识即可自动实现默认的序列化操作。

      ```java
      public class User implements Serializable{
           private static final long serialVersionUID = 871136882100083044L
           private String userName;
           private boolean isMale;
           ...
        』
      ```

      ```java
      //序列化操作
      User user = new User("Stephanie",false);
      ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("cache.txt"));
      out.writeObject(user);
      out.close

      //反序列化操作
      ObjectInputStream in = new ObjectInputStream(new FileInputStream("cache.txt"));
      User user = (User)in.readObject();
      in.close();
      ```

      > serialVersionUID工作机制：序列化时系统会将当前类的serialVersionUID写入序列化的文件中，当反序列化时系统会去检测文件中的serialVersionUID，看它是否和当前类的serialVersionUID一致，一致则说明版本相同可以成功反序列化；否则无法正常反序列化
      >
      > Notice: 
      >
      > 1. 静态成员变量属于类不属于对象，所以不会参与序列化过程；
      > 2. 用**transient**关键字标记的成员变量不参与序列化过程。
      > 3. 通过重写**readObject()**和**writeObject()**方法可以改变系统的默认序列化过程。
      
   2. Parcelable接口：Android提供的新的序列化方式。

      ```java
        //实现Parcelable典型用法
        public class User implements Parcelable{
            public int userId;
            public String userName;
            public boolean isMale;
            public Book book;
            //构造方法
            public User(int UserId,String userName,boolean isMale){
                this.userId = userId;
                this.userName = userName;
                this.isMale = isMale;
            }
            //返回当前对象的内容描述，含文件描述符返回1，否则返回0
            public int describeContents(){
                reaturn 0;
            }
            //将当前对象写入序列化结构中，flags标识有两种值(0,1),为1时标识当前对象需要作为返回值返回，不能     //立即释放资源
            public void writeToParcel(Parcel out,int flags){
                out.writeInt(userId);
                out.writeString(userName);
                out.writeInt(isMale ? 1 : 0);
                out.writeParcelable(book,0);
            }
            //实现反序列化功能
            public static final Parcelable.Creator<User> CREATER = new Parcelable.Creator<User>(){
                //从序列化后的对象中创建原始对象
                public User createrFromParcel(Parcel in){
                    return new User(in);
                }
                //创建指定长度的原始对象数组
                public User[] newArray(int size){
                    return new User[size];
                }
            };
            //
            private User(Parcel in){
                userId = in.readInt();
                userName = in.readString();
                isMale = in.readInt() == 1;
                book = in.readParcelable(Thread.currentThread().getContextClassLoader());
            }
        }
      ```
      > **Parcelable** 和**Serializable**选取：
      >
      > Serializable是Java中的序列化接口，使用简单但需要大量I/O操作，开销很大；Parcelable使用较麻烦但在Android平台上效率很高，所以首选**Parcelable**接口。

   3. Binder:Binder是Android中的一种跨进程通信方式。在Android开发中，Binder主要用于Service中，包括AIDL和Messenger。

      > 1. 当客户端发起远程请求时，由于当前线程会被挂起直至服务端进程返回数据，所以不能在UI线程中发起耗时的远程请求；
      > 2. 由于服务打的Binder方法运行在Binder的线程池中，所以Binder方法不管是否耗时都应该采用同步的方法去实现。

      ![图2-1Binder的工作机制](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图2-1Binder的工作机制.png)
      Binder实现步骤
      
      1. 声明一个AIDL性质的接口，只需继承**IInterface**接口
      2. 实现**Stub**类和Stub类的**Proxy**代理类
      
   4. 给Binder设置死亡代理：

      1. 声明一个**Deathecipient**对象。**Deathecipient**是一个接口，我们需要实现其内部的回调方法**binderDied**

         ```java
         private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient(){
             @Override
             public void binderDied(){
                 if(mBookManager == null)
                     return;
               mBookManager.asBinder().unlinkToDeath(mDeathRecipient,0);
               mBookManager = null;
             }
         }
         ```

      2. 在客户端绑定远程服务成功后，给binder设置死亡代理

         ```java
         mService  = IMessageBoxManager.Stub.asInterface(binder);
         binder.linkToDeath(mDeathRecipient,0);
         ```

      > 通过Binderd方法**isBinderAlive**可以判断Binder是否死亡        

4. ##### Android中IPC方式

   1. 使用Bundle

      四大组件中的三大组件(Activity，Service，Receiver)都是支持Intent中传递Bundle数据的，由于Bundle实现了Parcelable接口，所以它可以在不同的进程之间传输。但是，传输的数据必须能够被序列化，比如基本类型和实现了Parcelable接口的对象。

   2. 使用文件共享

      即两个进程通过**读/写**同一个文件来交换数据。除了可以交换一些文本信息外，还可以序列化一个对象到文件系统的同时另一个进程中恢复这个对象。

      ```java
      //MainActivity
      private void persistToFile(){
          new Thread(new Runnable(){
              @Override
              public void run(){
                  User user = new User(1,"hello world",false);
                  File dir = new File(MyConstants.CHAPTER_2_PATH);
                  if(!dir.exists()){
                      dir.mkdirs();
                  }
                  File cachedFile = new File(MyConstants.CACHE_FILE_PATH);
                  ObjectOutputStream objectOutputStream = null;
                  try{
                      objectOutputStream = new ObjectOutputStream(new                                                                FileOutputStream(cacheFile));
                      objectOutputStream.writeObject(user);
                      Log.d(TAG,"persist user : " + user);
                  }catch(IOException e){
                      e.printStackTrace();
                  }finally{
                      MyUtils.close(objectOutputStream);
                  }
              }
          }).start();
      }
      //SecondActivity
      private void recoverFromFile(){
          new Thread(new Runnable(){
              @Override
              public void run(){
                  User user = null;
                  File cachedFile = new File(MyConstants.CACHE_FILE_PATH);
                  if(cachedFile.exists()){
                      ObjectInputStream objectInputStream = null;
                      try{
                          objectInputStream = new ObjectInputStream(new FileInputStream(
                                                                       cachedFile));
                          user = (User) objectInputStream.readObject();
                          Log.d(TAG,"recover user :"+user);
                      }catch(IOException){
                          e.printStackTrace();
                      }catch(ClassNotFoundException e){
                          e.printStackTrace();
                      }finally{
                          MyUtils.close(objectInputStream);
                      }
                  }
              }
          }).start();
      }
      ```

      > 文件共享方式适合在对数据同步要求不高的进程之间进行通信，并且要妥善处理**并发读/写**的问题。

   3. 使用Messenger

      译为“信使”，可以在Message中放入需要传递的数据，通过它在不同进程中传递Message对象。它的底层实现是AIDL

      ```java
      //Messenger的构造方法
      public Messenger(Handler target){
          mTarget = target.getIMessenger();
      }
      public Messenger(IBinder target){
          mTarget = IMessenger.Stub.asInterface(target);
      }
      ```

      > Messenger一次处理一个请求，因此在服务端不用考虑线程同步的问题

      实现Messenger的步骤：

      1. 服务端进程

         首先在服务端创建一个Service来处理客户端的连接请求，同时创建一个Handler并通过它来创建    Messenger对象，然后在Service的onBind中返回Messenger对象底层的Binder。

      2. 客户端进程

         首先绑定服务端的Service，绑定成功后用服务端返回的IBinder对象创建一个Messenger，通过这个Messenger就可以向服务端发送Message对象。

         ```java
         //服务端
         public class MessengerService extends Service{
         private static final String TAG = "MessengerService";
         private static class MessengerHandler extend Handler{
             @Override
             public void handleMessage(Message msg){
                 switch(msg.what){
                   case MyConstants.MSG_FROM_CLIENT:
                     Log.i(TAG,"receive msg from client : "+msg.getData().getString("msg"));
                     break;
                   default:
                     super.handleMessage(msg);
                 }
             }
         }
         }
         ```

         ```xml
         //注册Service
         <service
                  android:name = "com.ryg.chapter_2.messenger.MessengerService"
                  android:process = ":remote">
         ```

         ```java
         //客户端
         public class MessengerActivity extends Activity{
             private static final String TAG = "MessengerActivity";
             private Messenger mService;
             private ServicerConnection mConnection = new ServiceConnection(){
                 public void onServiceConnected(ComponentName className,IBinder service){
                     mService = new Messenger(service);
                     Message msg = Message.obtain(null,"MyConstants.MSG_FROM_CLIENT");
                     Bundle data = new Bundle();
                     data.putString("msg","hello ,this is client.");
                     msg.setData(data);
                     try{
                         mService.send(msg);
                     }catch(RemoteException e){
                         e.printStackTrace();
                     }
                 }
                 public void onServiceDisconnected(ComponentName className){}
             };
             @Override
             protected void onCreate(Bundle savedInstanceState){
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_messenger);
                 Intent intent = new Intent(this,MessengerService.class);
                 bindService(intent,mConnection,Conatext.BIND_AUTO_CREATE);
             }
             @Override
             protected void onDestroy(){
                 unbindService(mConnection);
                 super.onDestroy();
             }
         }
         ```

   4. 使用AIDL

      AIDL的使用流程:

      1. 服务端

         - 创建一个Service用来监听客户端的连接
         - 创建一个AIDL文件声明给客户端的接口
         - 在Service中实现这个AIDL接口

      2. 客户端

         - 需要绑定服务端的Service
         - 绑定成功后将服务端返回的Binder对象转换成AIDL接口所属的类型

      3. AIDL接口的创建

         ```java
         //IBookManager.aidl
         package com.ryg.chapter_2.aidl
         import com.ryg.chapter_2.aidl.Book;
         interface IBookManager{
           List<Book> getBookList();
           void addBook(in Book book);
         }
         ```

         AIDL文件支持的数据类型

         - 基本数据类型(int , long , char , boolean , double等);
         - String和CharSequence;
         - List:只支持**ArrayList**,并且里面的每个元素都必须能够被AIDL支持；
         - Map:只支持**Hashap**，并且里面的每个元素都必须能够被AIDL支持；
         - Parcelabel：所有实现了Parcelable接口的对象；
         - AIDL：所有的AIDL接口本身也可以在AIDL文件中使用。

         > 自定义的Parcelable对象和AIDL对象必须要显式**import**进来，无论它是否与当前AIDL文件位于同一个包内。
         >
         > 如果AIDL文件中用到了自定义的Parcelable对象，必须新建一个和它同名的AIDL文件，并在其中声明它为Parcelabel类型。
         >
         > AIDL中除了基本数据类型，其他类型的参数必须标上方向：in，out，inout
         >
         > AIDL中的定向 tag 表示了在跨进程通信中数据的流向,数据流向是针对在客户端中的那个传入方法的对象而言的。
         >
         > ​         in： 表示数据只能由客户端流向服务端
         >
         > ​         out：表示数据只能由服务端流向客户端
         >
         > ​         inout：表示数据可在服务端与客户端之间双向流通     

      4. 远程服务端Service的实现

         ```java
         //服务端Service
         public class BookManagerService extends Service{
             private static final String TAG = "BMS";
             //CopyOnWriteArrayList:支持并发读/写。
             private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<Book>();
             private Binder mBinder = new IBookManager.Stub(){
                 @Override
                 public List<Book> getBookList() throws RemoteException{
                     return mBookList;
                 }
                 @Override
                 public void addBook(Book book)throws RemoteException{
                     mBookList.add(Book);
                 }
             };
             @Override
             public void onCreate(){
                 super.onCreate();
                 mBookList.add(new Book(1,"Android"));
                 mBookList.add(new Book(2,"IOS"));
             }
             @Override
             public IBinder onBind(Intent intent){
                 return mBinder;
         
             }
         }
         ```

         ```xml
         <service
                  android:name=".aidl.BookManagerService"
                  android:process=":remote"/>
         ```

      5. 客户端的实现

         ```java
         public class BookManagerActivity extends Activity{
             private static final String TAG = "BookManagerActivity";
             private ServiceConnection mConnection = new ServiceConnection(){
                 public void onServiceConnected(ComponentName className,IBinder service){
                     IBookManager bookManager = IBookManager.Stub.asInterface(service);
                     try{
                         List<Book> list = bookManager.getBookList();
                         Log.i(TAG,"query book list ,list"+                      
                                     "type:"+list.getClass().getCanonicalName());
                         Log.i(TAG,"query book list :"+list.toString());
                     }catch(RemoteException e){
                         e.printStackTrace();
                     }
                 }
                 public void onServiceDisconnected(ComponentName className){}
             };
             @Override
             protected void onCreate(Bundle savedInstanceState){
                 super.onCreate(savedInstanceState);
                 setContentView(R.layout.activity_book_manager);
                 Intent intent = new Intent(this,BookManagerService.class);
                 bindService(intent,mConnection,Context.BIND_AUTOCREATE);
             }
             @Override
             protected void onDestroy(){
                 unbindService(mConnection);
                 super.onDestroy();                                          
             }
         }
         ```
      
      6. 解除监听

         `RemoteCallbackList`是系统提供的用于删除跨进程Listener的接口

         ```java
      //声明RemoteCalbackList
         private RemoteCallbackList<IOnNewBookListener> mListenerList = new RemoteCallbackList<>();
         //修改注册和注销方法
         public void registerListener(IOnNewBookListener listener) throws RemoteException{
          mListener.register(listener);
         }
         public void unregisterListener(IOnNewBookListener listener) throws RemoteException{
             mListener.unregister(listener);
      }
         //通知Listener
      int N = mListenerList.beginBroadcast();
         for(int i = 0; i < N; i++) {
          IOnNewBookListener l = mListenerList.getBroadcastItem(i);
             if(l != null){
                 ......
             }
         }
         mListenerList.finishBroadcast();
         ```
   
         > RemoteCallbackList并不是一个List,遍历时必须按照上述流程进行，`beginBroadcast`和`finishBroadcast`必须要配对使用。
         >
         > 客户端的`onServiceConnected`和`onServiceDisconnected`方法都运行在主线程，不能在其中直接调用服务端的耗时方法。
         >
         > `onServiceDiscnnected`在客户端的UI线程中被回调，而`binderDied`在客户端的Binder线程池中被回调
   
      7. 在AIDL中进行权限验证
   
         - 在onBind中进行验证
         - 在服务端的onTransact方法中进行验证
   
   5. 使用ContentProvider
   
      ContentProvider是Android中提供的用于不同应用间进行数据共享的方式,其底层实现是Binder。
   
      自定义ContentProvider步骤：
   
      - 继承自ContentProvider类
      - 实现其中的抽象方法：onCreate,query,update,insert,delete和getType。
      - 注册这个类
   
      > 与query方法不同的是，update,insert和delete方法会引起数据源的改变，需要通过ContentResolver的**notifyhange**方法进行更新。
      >
      > 要观察一个ContentProvider中的数据变化，可以通过ContentResolver的registerContentObserver方法注册观察者，用unregisterContentObserver来进行解除。
   
   6. 使用Socket
   
      Socket，也称为“套接字”，它分为**流式套接字**和**用户数据报套接字**，分别对应与网络传输控制层中的**TCP**和**UDP**协议。两个进程可以通过Socket来实现信息的传输，Socket本身可以支持传输任意字节流。
   
      Socket连接过程
      
      ![](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/socket_connect.png)


5. ##### Binder连接池

   将每个业务模块的Binder请求统一转发到远程Service中去执行，从而避免了重复创建Service。

6. ##### 选用合适的IPC方式

   | 名称              | 优点                                | 缺点                                       | 适用场景                               |
   | --------------- | --------------------------------- | ---------------------------------------- | ---------------------------------- |
   | Bundle          | 简单易用                              | 只能传输Bundle支持的数据类型                        | 四大组件间的进程间通信                        |
   | 文件共享            | 简单易用                              | 不适合高并发场景，并且无法做到进程间的即时通信                  | 无并发访问情形，交换简单的数据实时性不高的场景            |
   | AIDL            | 功能强大                              | 使用稍复杂，需要处理好线程同步                          | 一对多通信且有RPC需求                       |
   | Messenger       | 功能一般，支持一对多串行通信，支持实时通信             | 不能很好处理高并发，不支持RPC，数据只能通过Message传输Bundle支持的数据类型 | 低并发的一对多即时通信，无RPC需求，或者无须要返回结果的RPC需求 |
   | ContentProvicer | 数据源访问功能强大，支持一对多并发数据共享，可通过Call方法扩展 | 可以理解为受约束的AIDL，主要提供数据源的CRUD操作             | 一对多的进程间的数据共享                       |
   | Socket          | 功能强大，可以通过网络传输字节流，支持一对多并发实时通信      | 实现细节稍繁琐，不支持直接的RPC                        | 网络数据交换                             |

   