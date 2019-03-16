### Android 开发艺术探索 第一章笔记

#### 第一章  Activity的生命周期和启动模式

##### Activity的生命周期  

1. ##### 典型情况下的生命周期 : 在用户参与的情况下，Activity所经过的生命周期的改变

   1. onCreate:Activity正在被创建。可以在这个方法中进行初始化工作，比如加载界面布局资源，初始化数据等。

   2. onRestart:Activity正在重新启动。

   3. onStart:Activity正在被启动，此时Activity已经**可见**，但是**没有出现在前台**，无法和用户进行交互。

   4. onResume:Activity已经可见，且**出现在前台开始活动**。

   5. onPause:Activity正在停止。

   6. onStop:Activity即将停止，可以做一些回收工作，但不能太耗时。

   7. onDestroy:Activity即将被销毁，生命周期最后一个回调，可以做一些回收工作和最终资源的释放。

      ![图1-1Activity生命周期切换过程](img\图1-1Activity生命周期切换过程.png)

   ps: (1) 一个Activity第一次启动时的回调为:onCreate->onStart->onResume

   ​      (2)打开新的Activity或切换到桌面时的回调为:onPause->onStop(如果新Activity采用了透明主题，则不会调用onStop)

   ​      (3)再次回到原Activity时的回调为:onRestart->onStart->onResume

   ​      (4)使用back键回退时的回调:onPause->onStop->onDestroy

   ###### Remarks:onStart和onStop是从Activity是否可见的角度进行回调的;onResume和onPause是从Activity是否位于前台的角度。       

2. ##### 异常情况下的生命周期 : Activity被系统回收或者当前设备的Configuration发生改变而导致Activity被销毁重建

   1. 资源相关的系统配置发生改变导致Activity被杀死并重新创建

   2. 资源内存不足导致低优先级的Activity被杀死

      Activity优先级情况(由高到低)

      - 前台Activity ——正在与用户交互的Activity，优先级最高
      - 可见但非前台Activity 
      - 后台Activity ——已经被暂停的Activity

##### Activity的启动模式

1. ##### Activity的LaunchMode

   1. standard:标准模式。每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否已经存在。*在这种模式下，谁启动了这个Activity，那么这个Activity就运行在启动它的那个Activity所在的栈中。*

   2. singleTop:栈顶复用模式。在这种模式下，如果新的Activity已经位于任务栈的栈顶，那么此Activity不会被重新创建，同时它的onNewIntent方法会被回调，并且这个Activity的onCreate,onStart不会被系统调用;如果新Activity的实例已存在但不是位于栈顶，那么新Activity仍然会重新创建。

   3. singleTask:栈内复用模式。这是一种单实例模式，在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例。

   4. singleInstance:单实例模式。除了具有singleTask模式的所有特性外，在这种模式下的Activity只能单独地位于一个任务栈中。

      *TaskAffinity:任务相关性。这个参数标识了一个Activity所需要的任务栈的名字，默认情况下为应用的包名。TaskAffinity属性主要和singleTask启动模式或者AllowTaskReparenting属性配对使用；任务栈分为前台任务栈和后台任务栈，后台任务栈中的Activity处于暂停状态。*

2. ##### 给Activity指定启动模式

   1. 通过**AndroidManifest**文件给Activity指定启动模式

      ```xml
      <activity
                android:name="com.ryg.chapter_1.secondActivity"
                android:configChanges="screenLayout"
                android:launchMode="singleTask"
                android:label="@string/app_name"/>
      ```

   2. 通过在**Intent**中设置标志位为Activity指定启动模式

      ```java
      Intent intent = new Intent();
      intent.setClass(MainActivity.this,SecondActivity.class);
      intent.addFlags(Intetn.FLAG_ACTIVITY_NEW_TASK);
      startActivity(intent);
      ```

      > 两种方式的区别：
      >
      > 1. 第二种的优先级要高于第一种，当两种同时存在时，以第二种方式为准；
      > 2. 第一种方式无法直接为Activity设定**FLAG_ACTIVITY_CLEAR_TOP**标识，而第二种方式无法为Activity指定**singlenstance**模式

3. ##### IntentFilter的匹配规则

   1. action(字符串)的匹配规则：**Intent**中的action必须能够和过滤规则中的action匹配，即action的字符串值完全一样。一个过滤规则中可以有多个action，只要**Intent**中的action能够和过滤规则中的任何一个action相同即可匹配成功。但如果**Intent**中没有指定action，则匹配失败，并且action区分大小写。
   2. category(字符串)的匹配规则：它要求**Intent**中如果含有category，那么所有的category都必须和过滤规则中的其中一个category相同。如果Intent中没有，任然可以匹配成功。
   3. data的匹配规则
      1. 结构：由*mimeType*和*URI*两部分组成。其中*mimeType*指的是媒体类型，*URI*是资源地址。
      2. 匹配规则：