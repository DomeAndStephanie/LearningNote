[TOC]

#### Service知识总结

---

##### 一. 概述

Service是一种可在后台执行长时间运行操作而不提供界面的应用组件。服务可由其他应用组件启动，而且即使用户切换到其他应用，服务仍将在后台继续运行。此外，组件可通过绑定到服务与之进行交互，执行进程间通信 (IPC)。

服务可分为三种类型：

- 前台服务：前台服务执行一些用户能注意到的操作，必须显示**通知**。即使用户停止与应用的交互，前台服务仍会继续运行
- 后台服务：后台服务执行用户不会直接注意到的操作。(API26或更高时，系统会对运行后台服务施加限制，不包括绑定服务)
- 绑定服务：应用组件通过`bindService`绑定的服务。绑定服务会提供客户端-服务器接口，以便组件与服务进行交互、发送请求、接收结果，甚至是利用进程间通信 (IPC) 跨进程执行这些操作。仅当与另一个应用组件绑定时，绑定服务才会运行。多个组件可同时绑定到该服务，但全部取消绑定后，该服务即会被销毁。

> 服务可以同时以**启动服务**和**绑定服务**两种方式运行。
>
> 服务在其托管进程的主线程中运行，它既**不**创建自己的线程，也**不**在单独的进程中运行（除非另行指定）。

##### 二. 生命周期

![](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/service_lifecycle.png)

- onCreate:首次创建服务时，系统会（在调用` onStartCommand()`或`onBind()`）调用此方法来执行一次性设置程序。如果服务已在运行，则不会调用此方法。

- onStartCommand:`启动服务方式`回调，当另一个组件（如 `Activity`）请求启动服务时，系统会通过调用 startService() 来调用此方法。执行此方法时，服务即会启动并可在后台无限期运行。`onStartCommand()` 方法必须返回整型数。整型数是一个值，用于描述系统应如何在系统终止服务的情况下继续运行服务。

  > ```
  > START_NOT_STICKY
  > ```
  >
  > 如果系统在 `onStartCommand()` 返回后终止服务，则除非有待传递的挂起 Intent，否则系统*不会*重建服务。这是最安全的选项，可以避免在不必要时以及应用能够轻松重启所有未完成的作业时运行服务。
  >
  > ```
  > START_STICKY
  > ```
  >
  > 如果系统在 `onStartCommand()` 返回后终止服务，则其会重建服务并调用 `onStartCommand()`，但*不会*重新传递最后一个 Intent。相反，除非有挂起 Intent 要启动服务，否则系统会调用包含空 Intent 的 `onStartCommand()`。在此情况下，系统会传递这些 Intent。此常量适用于不执行命令、但无限期运行并等待作业的媒体播放器（或类似服务）。
  >
  > ```
  > START_REDELIVER_INTENT
  > ```
  >
  > 如果系统在 `onStartCommand()` 返回后终止服务，则其会重建服务，并通过传递给服务的最后一个 Intent 调用 `onStartCommand()`。所有挂起 Intent 均依次传递。此常量适用于主动执行应立即恢复的作业（例如下载文件）的服务。

- onBind:`绑定服务方式`回调，当另一个组件想要与服务绑定时，系统会通过调用 `bindService()` 来调用此方法。在此方法的实现中，您必须通过返回 `IBinder` 提供一个接口，以供客户端用来与服务进行通信。；如果并不希望允许绑定，则应返回 null。

- onUnbind:`绑定服务方式`回调，

- onDestroy:当不再使用服务且准备将其销毁时，系统会调用此方法。可以通过实现此方法来清理任何资源，如线程、注册的侦听器、接收器等。这是服务接收的最后一个调用。

  ![](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/service_binding_tree_lifecycle.png)

  <p align="center">已启动且允许绑定的服务的生命周期</p>

##### 三.使用方式

- 启动服务方式

  1. 定义服务类(创建`Service`的子类)

     ```java
     public class ExampleService extends Service {
     
       @Override
       public void onCreate() {
       }
     
       @Override
       public int onStartCommand(Intent intent, int flags, int startId) {
           Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
           // If we get killed, after returning from here, restart
           return START_STICKY;
       }
     
       @Override
       public IBinder onBind(Intent intent) {
           // We don't provide binding, so return null
           return null;
       }
     
       @Override
       public void onDestroy() {
         Toast.makeText(this, "service done", Toast.LENGTH_SHORT).show();
       }
     }
     ```

  2. 在manifest文件中进行注册

     ```xml
     <manifest ... >
       ...
       <application ... >
           <service android:name=".ExampleService" />
           ...
       </application>
     </manifest>
     ```

  3. 启动服务

     ```java
     Intent intent = new Intent(this, ExampleService.class);
     startService(intent);
     ```

  4. 停止服务

     服务必须通过调用 `stopSelf() `自行停止运行，或由另一个组件通过调用 `stopService() `来停止它。

- 绑定服务方式

  绑定服务是客户端-服务器接口中的服务器。借助绑定服务，组件（例如 Activity）可以绑定到服务、发送请求、接收响应，以及执行进程间通信 (IPC)。绑定服务通常只在为其他应用组件提供服务时处于活动状态，不会无限期在后台运行。

  1. 创建服务类

     与启动服务相同，不同点在于需通过实现 `onBind()` 回调方法返回 `IBinder`，从而定义与服务进行通信的接口。然后，其他应用组件可通过调用 `bindService()` 来检索该接口，并开始调用与服务相关的方法。

  2. 在manifest中进行注册

  3. 绑定服务

     ```java
     Intent intent = new Intent(this, LocalService.class);
     bindService(intent, connection, Context.BIND_AUTO_CREATE);
     ```

     **intent**:显式命名要绑定的服务,使用隐式Intent存在安全隐患，无法确定哪些服务会响应该Intent,在API21以上时使用隐式Intent绑定服务会抛出异常

     **connection**:实现 `ServiceConnection`重写两个回调方法：

     `onServiceConnected()`:系统会调用该方法，进而传递服务的 `onBind()` 方法所返回的 `IBinder`

     `onServiceDisconnected()`:当与服务的连接意外中断（例如服务崩溃或被终止）时，Android 系统会调用该方法。当客户端取消绑定时，系统*不会*调用该方法。

     **Context.BIND_AUTO_CREATE**：示绑定选项的标记

  4. 解绑服务

  创建提供绑定的服务时，必须提供 `IBinder`，进而提供编程接口，以便客户端使用此接口与服务进行交互。可以通过三种方法定义接口：

  - 扩展 `Binder `类

    如果服务是供您的自有应用专用，并且在与客户端相同的进程中运行（常见情况），则应通过扩展 `Binder` 类并从 `onBind()` 返回该类的实例来创建接口。收到 `Binder` 后，客户端可利用其直接访问 `Binder` 实现或 `Service` 中可用的公共方法。

    具体方法：

    1. 创建服务类，并在服务中创建可执行以下某种操作的 `Binder` 实例：
       - 包含客户端可调用的公共方法。
       - 返回当前的 `Service` 实例，该实例中包含客户端可调用的公共方法。
       - 返回由服务承载的其他类的实例，其中包含客户端可调用的公共方法。
    2. 从 `onBind()` 回调方法返回此 `Binder` 实例。
    3. 在客户端中，从 `onServiceConnected()` 回调方法接收 `Binder`，并使用提供的方法调用绑定服务。

  - 使用`Messenger`

    如需让接口跨不同进程工作，可以使用 `Messenger` 为服务创建接口。服务可借此方式定义 `Handler`，以响应不同类型的 `Message` 对象。此 `Handler` 是 `Messenger` 的基础，后者随后可与客户端分享一个 `IBinder`，以便客户端能利用 `Message` 对象向服务发送命令。此外，客户端还可定义自有 `Messenger`，以便服务回传消息。

    具体方法：

    1. 服务实现一个 `Handler`，由该类为每个客户端调用接收回调。
    2. 服务使用 `Handler` 来创建 `Messenger` 对象（对 `Handler` 的引用）。
    3. `Messenger` 创建一个 `IBinder`，服务通过 `onBind()` 使其返回客户端。
    4. 客户端使用 `IBinder` 将 `Messenger`（其引用服务的 `Handler`）实例化，然后使用后者将 `Message` 对象发送给服务。
    5. 服务在其 `Handler` 中（具体地讲，是在 `handleMessage()` 方法中）接收每个 `Message`。

  - 使用`AIDL`

    Android 接口定义语言 (AIDL) 会将对象分解成原语，操作系统可通过识别这些原语并将其编组到各进程中来执行 IPC。

    具体方法：

    1. 创建` .aidl `文件,此文件定义带有方法签名的编程接口。
    2. 实现接口，Android SDK 工具会基于 `.aidl` 文件，使用 Java 编程语言自动生成接口。
    3. 向客户端公开接口，实现 `Service` 并重写 `onBind()`，从而返回 `Stub` 类的实现。

##### 四. 启动过程

见[Android开发艺术探索第九章#Service的工作过程](../../Android开发艺术探索/Android开发艺术探索第九章笔记.md)

![](http://gityuan.com/images/android-service/start_service/start_service_processes.jpg)

<p align="center">图引自http://gityuan.com/2016/03/06/start-service/,若侵即删</p>

##### 五. 工作机制

##### 补充

###### 1.前台服务

- 前台服务执行用户可以注意到的操作，且每个前台服务必须显示一个状态栏通知

- 使用方式

  1. 申请权限(API28之后必须请求该权限，该权限是一个普通权限)

     ```xml
     <manifest xmlns:android="http://schemas.android.com/apk/res/android" ...>
         <uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
     
         <application ...>
             ...
         </application>
     </manifest>
     ```

  2. 启动服务

     ```java
     Intent notificationIntent = new Intent(this, ExampleActivity.class);
     PendingIntent pendingIntent =
         PendingIntent.getActivity(this, 0, notificationIntent, 0);
     
     Notification notification =
         new Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
         .setContentTitle(getText(R.string.notification_title))
         .setContentText(getText(R.string.notification_message))
         .setSmallIcon(R.drawable.icon)
         .setContentIntent(pendingIntent)
         .setTicker(getText(R.string.ticker_text))
         .build();
     
     // Notification ID cannot be 0.
     startForeground(ONGOING_NOTIFICATION_ID, notification);
     ```

###### 2.Service与Thread的区别

1. Thread是程序执行的最小单元，是分配CPU的基本单位，可以执行异步操作；Service是Android的一种机制，依托在所在的主线程运行
2. Thread的运行是独立的，Service绑定该应用.

###### 3.IntentService

详情见[IntentService](../其它/IntentService.md)