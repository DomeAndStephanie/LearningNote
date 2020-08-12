### Android 开发艺术探索 第一章笔记

#### 第一章  Activity 的生命周期和启动模式

##### Activity 的生命周期

1. ##### 典型情况下的生命周期 : 在用户参与的情况下，Activity 所经过的生命周期的改变

   1. onRestart:Activity 正在重新启动。
   
   2. onCreate:Activity 正在被创建。可以在这个方法中进行初始化工作，比如加载界面布局资源，初始化数据等。
   
   3. onStart:Activity 正在被启动，此时 Activity 已经**可见**，但是**没有出现在前台**，无法和用户进行交互。
   
   4. onResume:Activity 已经可见，且**出现在前台开始活动**。

   5. onPause:Activity 正在停止。

   6. onStop:Activity 即将停止，可以做一些回收工作，但不能太耗时。
   
   7. onDestroy:Activity 即将被销毁，生命周期最后一个回调，可以做一些回收工作和最终资源的释放。
   
      ![图1-1Activity生命周期切换过程](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图2-1Binder的工作机制.png)
   
   > (1) 一个 Activity 第一次启动时的回调为:onCreate->onStart->onResume
  >
   > (2)打开新的 Activity 或切换到桌面时的回调为:onPause->onStop(如果新 Activity 采用了透明主题，则不会调用 onStop)
>
   > (3)再次回到原 Activity 时的回调为:onRestart->onStart->onResume
>
   > (4)使用 back 键回退时的回调:onPause->onStop->onDestroy


   ###### Remarks:onStart 和 onStop 是从 Activity 是否可见的角度进行回调的;onResume 和 onPause 是从 Activity 是否位于前台的角度。
2. ##### 异常情况下的生命周期 : Activity 被系统回收或者当前设备的 Configuration 发生改变而导致 Activity 被销毁重建

   1. 资源相关的系统配置发生改变导致 Activity 被杀死并重新创建
   2. 资源内存不足导致低优先级的 Activity 被杀死

      Activity 优先级情况(由高到低)

      - 前台 Activity ——正在与用户交互的 Activity，优先级最高
      - 可见但非前台 Activity
      - 后台 Activity ——已经被暂停的 Activity

##### Activity 的启动模式

1. ##### Activity 的 LaunchMode

   1. **standard**:标准模式。每次启动一个 Activity 都会重新创建一个新的实例，不管这个实例是否已经存在。*在这种模式下，谁启动了这个 Activity，那么这个 Activity 就运行在启动它的那个 Activity 所在的栈中。*
   2. **singleTop**:栈顶复用模式。在这种模式下，如果新的 Activity 已经位于任务栈的栈顶，那么此 Activity 不会被重新创建，同时它的 onNewIntent 方法会被回调，并且这个 Activity 的 onCreate,onStart 不会被系统调用;如果新 Activity 的实例已存在但不是位于栈顶，那么新 Activity 仍然会重新创建。
   3. **singleTask**:栈内复用模式。这是一种单实例模式，在这种模式下，只要 Activity 在一个栈中存在，那么多次启动此 Activity 都不会重新创建实例。
   4. **singleInstance**:单实例模式。除了具有 singleTask 模式的所有特性外，在这种模式下的 Activity 只能单独地位于一个任务栈中。

      *TaskAffinity:任务相关性。这个参数标识了一个 Activity 所需要的任务栈的名字，默认情况下为应用的包名。TaskAffinity 属性主要和 singleTask 启动模式或者 AllowTaskReparenting 属性配对使用；任务栈分为前台任务栈和后台任务栈，后台任务栈中的 Activity 处于暂停状态。*
   
2. ##### 给 Activity 指定启动模式

   1. 通过 **AndroidManifest** 文件给 Activity 指定启动模式

      ```xml
      <activity
                android:name="com.ryg.chapter_1.secondActivity"
                android:configChanges="screenLayout"
                android:launchMode="singleTask"
                android:label="@string/app_name"/>
      ```
   2. 通过在 **Intent** 中设置标志位为 Activity 指定启动模式

      ```java
      Intent intent = new Intent();
      intent.setClass(MainActivity.this,SecondActivity.class);
      intent.addFlags(Intetn.FLAG_ACTIVITY_NEW_TASK);
      startActivity(intent);
      ```

      > 两种方式的区别：
      >
      > 1. 第二种的优先级要高于第一种，当两种同时存在时，以第二种方式为准；
      > 2. 第一种方式无法直接为 Activity 设定 **FLAG_ACTIVITY_CLEAR_TOP** 标识，而第二种方式无法为 Activity 指定 **singlenstance** 模式
      >
   
3. ##### Activity的Flags

   - FLAG_ACTIVITY_NEW_TASK:为Activity指定“singleTask”启动模式
   - FLAG_ACTIVITY_SINGLE_TOP:为Activity指定“singleTop”启动模式
   - FLAG_ACTIVITY_CLEAR_TOP:具有此标记位的Activity启动时，在同一个任务栈中所有位于它上面的Activity都要出栈
   - FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS:具有此标记的Activity不会出现在历史Activity的列表中。

4. ##### IntentFilter 的匹配规则

   1. action(字符串)的匹配规则：**Intent** 中的 action 必须能够和过滤规则中的 action 匹配，即 action 的字符串值完全一样。一个过滤规则中可以有多个 action，只要 **Intent** 中的 action 能够和过滤规则中的任何一个 action 相同即可匹配成功。但如果 **Intent** 中没有指定 action，则匹配失败，并且 action 区分大小写。
   2. category(字符串)的匹配规则：它要求 **Intent** 中如果含有 category，那么所有的 category 都必须和过滤规则中的其中一个 category 相同。如果 Intent 中没有，仍然可以匹配成功。
   3. data 的匹配规则
      1. 结构：由 *mimeType* 和 *URI* 两部分组成。其中 *mimeType* 指的是媒体类型，*URI* 是资源地址。
      
         ```xml
         <!-- 语法 -->
         <data android:scheme="string"
               android:host="string"
               android:port="string"
               android:path="string"
               andrid:pathPattern="string"
               android:pathPrefix="string"
               android:mimeType="string" />
         <!-- 结构 -->
         <scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
         ```
      
      2. 匹配规则：Intent中必须含有data数据，并且data数据能够完全匹配过滤规则中的某一个data
