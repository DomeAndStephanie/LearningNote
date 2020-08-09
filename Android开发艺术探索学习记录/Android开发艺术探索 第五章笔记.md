### 理解RemoteViews



#### 1. RemoteViews的应用

RemoteViews在实际开发中主要用在通知栏和桌面小部件的开发中。通知栏主要通过`NotificationManager`的**notify**方法实现，桌面小部件则通过`AppWidgetProvider`实现。这两个的界面都运行在系统的**SystemServer**进程中，无法直接更新View.

##### 	1.RemoteViews在通知栏上的应用

- 使用系统默认样式

  ```java
  // Build the notification and add the action.
  Notification newMessageNotification = new Notification.Builder(context, CHANNEL_ID)
          .setSmallIcon(R.drawable.ic_message)
          .setContentTitle(getString(R.string.title))
          .setContentText(getString(R.string.content))
          .addAction(action)
          .build();
  
  // Issue the notification.
  NotificationManagerCompat notificationManager = NotificationManagerCompat.from(this);
  notificationManager.notify(notificationId, newMessageNotification);
  ```

- 自定义通知

  ```java
  // Get the layouts to use in the custom notification
  RemoteViews notificationLayout = new RemoteViews(getPackageName(), R.layout.notification_small);
  RemoteViews notificationLayoutExpanded = new RemoteViews(getPackageName(), R.layout.notification_large);
  
  // Apply the layouts to the notification
  Notification customNotification = new NotificationCompat.Builder(context, CHANNEL_ID)
          .setSmallIcon(R.drawable.notification_icon)
          .setStyle(new NotificationCompat.DecoratedCustomViewStyle())
          .setCustomContentView(notificationLayout)
          .setCustomBigContentView(notificationLayoutExpanded)
          .build();
  ```

#####     2.RemoteViews在桌面小部件上的应用

 		AppWidgetProvider是Android中提供的用于实现桌面小部件的类，本质是一个广播

- 定义小部件界面

  在res/layout/下建立小部件布局界面

- 定义小部件配置信息

  在res/xml/下建立小部件配置信息

  > 重要参数说明：
  >
  > initLayout:初始化界面
  >
  > updatePeriodMillis:自动更新周期

- 定义小部件的实现类

  继承AppWidgetProvider实现具体逻辑

  > 重要方法说明：
  >
  > onEnable:小部件第一次添加到桌面时调用该方法
  >
  > onUpdate:小部件被添加时或者每次小部件更新时都会调用该方法
  >
  > onDeleted:每删除一次桌面小部件就调用一次
  >
  > onDisabled:最后一个该类型的桌面小部件被删除时调用该方法
  >
  > onReceive:分发具体的事件给其他方法

- 在AndroidManifest.xml中声明小部件

  #####     3.PendingIntent概述

​		PendingIntent表示一种处于待定，等待，即将发生状态的意图，支持三种待定意图：启动Activity,启动Service,发送		广播

```java
getActivity(Context context, int requestCode, Intent intent, int flags);
getService(Context context, int requestCode, Intent intent, int flags);
getBroadcast(Context context, int requestCode, Intent intent, int flags);
```

- 匹配规则：如果两个PendingIntent它们内部的`Intent`相同并且`requestCode`也相同，则这两个PendingIntent相同

  > Intent匹配规则：如果两个Intent的`ComponentName`和`intent-filter`都相同，则这两个Intent相同

- Flag参数

  - FLAG_ONE_SHOT

    当前PendingIntent只能使用一次

  - FLAG_NO_CREATE

    当前PendingIntent不会主动创建，如果不存在，则get方法会直接返回`null`

  - FLAG_CANCEL_CURRENT

    当前描述的PendingIntent如果已存在，那么它们都会被`cancel`

  - FLAG_UPDATE_CURRENT

    当前描述的PendingIntent如果已存在，那么它们都会被更新

#### 2.RemoteViews的内部机制

- 构造方法

  ```java
  /**
  * @param packageName 当前应用的包名
  * @param layoutId 待加载的布局文件
  **/
  public RemoteViews(String packageName, int layoutId)
  ```

  > RemoteViews目前并不支持所有的View类型，支持的所有类型如下
  >
  > Layout : FrameLayout, LinearLayout, RelativeLayout, GridLayout
  >
  > View : AnalogClock, Button, Chronometer, ImageButton, ImageView, ProgressBar, TextView, ViewFlipper, ListView, GridView, StackView, AdapterViewFlipper, ViewStub

- 使用

  RemoteViews没有提供findViewById方法，无法直接访问View元素，必须通过RemoteViews所提供的一系列`set`方法来完成

- 内部机制

  1. RemoteViews会通过Binder传递到SystemServer进程

  2. 系统根据RemoteViews中的包名等信息得到该应用的资源，加载RemoteViews中的文件

  3. 系统对生成的View进行一系列界面更新任务

     ![](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图5-1RemoteViews内部机制.png)

     > 单击事件，RemoteViews中只支持发起PendingIntent,不支持onClickListener.
     >
     > setOnClickPendingIntent用于给普通View设置单击事件，但不能给集合(ListView,StackView)中的View设置单击事件
     >
     > 要给ListView或StackView中的item添加单击事件，必须将setPendingIntentTemplate和setOnClickFillInIntent组合使用





