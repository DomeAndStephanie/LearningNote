### 第九章　四大组件的工作过程

1. #### 四大组件的运行状态

   - Activity

     展示一个界面并和用户进行交互．由**Intent**触发启动 ，分为显示Intent和隐式Intent，显示Intent可以明确指向一个Activity组件，隐式Intent可以指向**一个或多个**Activity组件

   - Service

     一种计算型组件，用于在后台执行一系列计算．与Activity只有一种启动状态不同，Service有两种状态：启动状态（startService）和绑定状态（bindService）．Service是运行在主线程中的．

   - BroadcastReceiver

     一种消息型组件，用于在不同的组件，不同的应用之间传递消息．

     广播有两种注册方式：

     1. 静态注册：在AndroidManifest中注册广播，在应用安装时被系统解析，无须应用启动就可以收到相应的广播．
     2. 动态注册：通过**Context.registerReceiver()**和**Context.unRegisterReceiver()**来注册和注销广播，需要应用启动才能注册并接收广播．

   - ContentProvider

     一种数据共享型组件，用于向其它组件和应用共享数据．

2. #### Activity的工作过程

   - 显示调用，启动Activity

     ```java
     Intent intent = new Intent(this, TestActivity.Class);
     startActivity(intent);
     ```

   - 启动流程

     ```mermaid
     sequenceDiagram
     Activity ->> Activity : startActvity
     Activity ->> Activity : startActivityForResult
     Activity ->> Instrumentation : execStartActivity
     Instrumentation ->> ActivityManagerService : ActivityManagerNative.getDefault().startActivity
     ActivityManagerService ->> ActivityManagerService : startActivity
     ActivityManagerService ->> ActivityStackSupervisor : startActivityLocked
     ActivityStackSupervisor ->> ActivityStack : resumeTopActivitiesLocked
     ActivityStack ->> ActivityStackSupervisor : startSpecificActivityLocked
     ActivityStackSupervisor ->> ActivityStackSupervisor : realStartActivityLocked
     ActivityStackSupervisor ->> ApplicationThread : sceduleLaunchActivity
     ApplicationThread ->> ActivityThread : handleLaunchActivity
     ActivityThread ->> ActivityThread : performLaunchActivity
     ```

     `performLaunchActivity`方法具体内容：

     1. 从**ActivityClientRecord**中获取待启动的Activity的组件信息
     2. 通过**Instrumentation**的*newActivity*方法使用类加载器创建Activity对象
     3. 通过**LoadedApk**的*makeApplication*方法来尝试创建Application对象
     4. 创建**ContextImpl**对象并通过Activity的*attach*方法完成h重要数据初始化
     5. 调用Activity的*onCreate*方法

1. #### Service的工作过程

   - 启动状态：主要用于执行后台计算

     ```java
     //启动Service
     Intent intentService = new Intent(this,TmpService.class);
     startService(intentService);
     ```

   - 绑定状态：用于其他组件和Service的交互

     ```java
     //绑定Service
     Intent intentService = new Intent(this,TmpService.class);
     bindService(intentService,mServiceConn,BIND_AUTO_CREATE);
     ```

   > 两种状态可以共存，一个Service既可以处于启动状态，也可以同时处于绑定状态

   1. Service的启动过程
   
      ```mermaid
      sequenceDiagram
      Activity ->> ContextWrapper : startService
      ContextWrapper ->> ContextImpl : startService
      ContextImpl ->> ContextImpl : startServiceCommon
      ContextImpl ->> ActivityManagerService : startService
      ActivityManagerService ->> ActiveServices : startServiceLocked
      ActiveServices ->> ActiveServices : startServiceInnerLocked
      ActiveServices ->> ActiveServices : bringUpServiceLocked
      ActiveServices ->> ApplicationThread : realStartServiceLocked
      ApplicationThread ->> ApplicationThread : scheduleCreateService
      ApplicationThread ->> ActivityThread : handleCreateService
      ```
   
      `handleCreateService`方法具体内容：
   
      1. 通过类加载器创建**Service**的实例
      2. 创建**Application**对象并调用其*onCreate*
      3. 创建**ContextImpl**对象并通过**Service**的*attach*方法建立联系
      4. 调用**Service**的*onCreate*方法并将**Service**对象存到**ActivityThread**的列表中
   
   2. Service的绑定过程
   
      ```mermaid
      sequenceDiagram
      ContextWrapper ->> ContextImpl : bindService
      ContextImpl ->> ContextImpl : bindServiceCommon
      ContextImpl ->> LoadedApk : getServiceDispatcher
      LoadedApk -->> ContextImpl : ServiceDispatcher.InnerConnection
      ContextImpl ->> ActivityManagerService : bindService
      ActivityManagerService ->> ActiveServices : bindServiceLocked
      ActiveServices ->> ActiveServices : bringUpServiceLocked
      ActiveServices ->> ApplicationThread : realStartServiceLocked
      ApplicationThread ->> ActivityManagerService : handleBindService
      ActivityManagerService ->> ActivityManagerService : publishService
      ActivityManagerService ->> ActiveServices : publishServiceLocked
      ```
   
4. #### BroadcastReceiver的工作过程

   1. 广播的注册过程

      - 静态注册

        静态注册的广播在应用安装时由`PackageManagerService`来完成整个注册过程(其它三大组件亦是如此)

      - 动态注册

        ```mermaid
        sequenceDiagram
        ContextWrapper ->> ContextImpl : registerReceiver
        ContextImpl ->> ContextImpl : registerReceiverInternal
        ContextImpl ->> LoadedApk : getReceiverDispatcher
        LoadedApk -->> ContextImpl : IIntentReceiver
        ContextImpl ->> ActivityManagerService : registerReceiver
        ```

   2. 广播的发送和接收过程

      - 广播的发送过程

        ```mermaid
        sequenceDiagram
        ContextWrapper ->> ContextImpl : sendBroadcast
        ContextImpl ->> ActivityManagerService : broadcastIntent
        ActivityManagerService ->> ActivityManagerService : broadcastIntentLocked
        ActivityManagerService ->> ActivityManagerService : scheduleBroadcastsLocked
        ActivityManagerService ->> ActivityManagerService : processNextBroadcast
        ActivityManagerService ->> ActivityManagerService : performReceiveLocked
        ActivityManagerService ->> ApplicationThread : scheduleRegisteredReceiver
        ApplicationThread ->> InnerReceiver : performReceiver
        InnerReceiver ->> LoadedApk.ReceiverDispatcher : performReceive
        ```

        > 应用处于停止状态的两种情形：
        >
        > 1. 应用安装后未运行
        > 2. 应用被手动或其他应用强制停止

5. #### ContentProvider的工作过程

   **ContentProvider**是一种内容共享型组件，它通过*Binder*向其他组件乃至其他应用提供数据。当**ContentProvider**所在的进程启动时，**ContentProvider**会同时启动并被发布到**AMS**中。这时候**ContentProvider**的*onCreate*要先于**Application**的*onCreate*而执行
   ![image-20200802104139887](D:\Downloads\图片\图9-1 ContentProvider的启动过程.png)
   
   ContentProvider启动后，外界可以提供它所提供的**增删改查**四个接口来操作数据源，这四个方法都是通过Binder来调用的，外界无法直接访问ContentProvider，只能通过ActivityManagerService根据Uri来获取对应的ContentProvider的Binder接口IContentProvider，再通过IContentProvider访问ContentProvider中的数据源
   
   - ContentProvider的启动过程(以query为例)
   
     ```mermaid
     sequenceDiagram
     ActivityThread ->> ActivityManagerService : startProcessLocked
     ActivityManagerService ->> ActivityThread : main
     ActivityThread ->> ActivityThread : attach
     ActivityThread ->> ActivityManagerService : attachApplication
     ActivityManagerService ->> ActivityManagerService : attachApplicationLocked
     ActivityManagerService ->> ApplicationThread : bindApplication
     ApplicationThread ->> ActivityThread : handleBindApplication
     ```
   
     `handleBindApplication`具体内容：
   
     1. 创建**ContextImpl**和**Instrumentation**
     2. 创建**Application**对象
     3. 启动当前进程的**ContentProvider**并调用其*onCreate*f方法
     4. 调用**Application**的*onCreate*方法



