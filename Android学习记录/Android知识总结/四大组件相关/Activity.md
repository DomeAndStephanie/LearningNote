[TOC]

#### Activity知识总结

##### 一. 定义

- 概念

  Activity是应用与用户互动的接口，提供窗口供应用在其中绘制界面

- 使用

  1. 创建继承Activity的自定义类，实现界面显示功能

     ```Kotlin
     class MainActivity : AppCompatActivity() {
         ....
     }
     ```

  2. 配置清单

     ```xml
     <!-- 声明Activity -->
     <maniest>
         <application>
             <activity android:name="{自定义类类名}"
                       android:permission="{自定义权限}">
             	<!-- 声明intent过滤器-->
                 <intent-filter>
                     <action android:name="{自定义action}" />
                     <category android:name="xxx" />
                     <data android:mimeType="xxx" />
                 </intent-filter>
             </activity>
         </application>
     </maniest>
     ```

     > 借助Intent过滤器，既可以根据显式请求启动Activity,也可以通过隐式请求启动Activity

##### 二. 生命周期

![](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图1-1Activity生命周期切换过程.png)

###### Activity的四种状态

> active:运行状态，当前Activity获取焦点
>
> paused:暂停状态，当前Activity失去焦点，仍然可见
>
> stopped:不可见但成员变量等依然存在
>
> killed:被销毁，成员变量等被收回

###### Activity的生命周期

- 生命周期方法含义
  1. onRestart:Activity 正在重新启动。
  2. onCreate:Activity 正在被创建。可以在这个方法中进行初始化工作，比如加载界面布局资源，初始化数据等。
  3. onStart:Activity 正在被启动，此时 Activity 已经***\*可见\****，但是***\*没有出现在前台\****，无法和用户进行交互。
  4. onResume:Activity 已经可见，且***\*出现在前台开始活动\****。
  5. onPause:Activity 正在停止。
  6. onStop:Activity 即将停止，可以做一些回收工作，但不能太耗时。
  7. onDestroy:Activity 即将被销毁，生命周期最后一个回调，可以做一些回收工作和最终资源的释放。

- 正常情况下的声明周期

  1. 一个 Activity 第一次启动时的回调为:onCreate->onStart->onResume
  2. *打开新的 Activity 或切换到桌面时的回调为:onPause->onStop(如果新 Activity 采用了透明主题，则不会调用 onStop)*
  3. 再次回到原 Activity 时的回调为:onRestart->onStart->onResume
  4. *使用 back 键回退时的回调:onPause->onStop->onDestroy*

- Dialog弹出时

  1. 如果是创建的dialog,由于依附的是当前的Activity,不会执行生命周期方法
  2. 跳转到一个不是全屏的Activity：*onPause -> onStop*

- 横竖屏切换时

  1. 不设置`android:configChanges`，切屏会重新调用各个生命周期，切横屏时执行一次，切竖屏时执行两次
  2. 设置`android:configChanges=“orientation"`时，切屏还是会重新调用各个生命周期，横屏和竖屏只会执行一次
  3. 设置`android:configChanges="orientation|keyboardHidden"`时，切屏不会调用各个生命周期，只会执行`onConfigurationChanged`方法

- 锁屏与解锁时

  锁屏时只会调用*onPause*，不会调用*onStop*方法，解锁后调用*onResume*
  
- 异常退出

  - Activity异常退出的生命周期：*onPause* --> *onSaveInstanceState* --> *onStop* --> *onDestroy*

    > *onPause*与*onSaveInstanceState*没有严格的先后关系，可能在*onPause*前，也可能在其后，但会在*onStop*之前调用

  - 异常退出又重新启动：*onCreate* --> *onStart* --> *onRestoreInstanceState* --> *onResume*

  - 如果要恢复原有Activity,则在*onSaveInstanceState*中保存数据，在*onRestoreInstanceState*中进行恢复。

##### 三. 任务栈与启动模式

###### 任务栈

- 任务：用户在执行某项工作时与之互动的一系列Activity集合

  每个Activity都是相互独立的界面，通过任务这个概念多个Activity关联组成一个应用。应用中的入口Activity作为任务中的根Activity。

- 返回栈(任务栈)

  返回栈是任务的实际载体，每个任务中所有的Activity都会按照各自的打开顺序保存在对应的返回栈中。由于是栈结构(先入后出)，外部只能访问栈顶的Activity,所以一个Activity要显示就必须在栈顶

- 进栈与出栈

  从当前Activity启动另一个Activity时，新的Activity被推送至栈顶，获取到焦点显示在屏幕上，原来的Activity仍保留在栈中，处于停止状态

  当用户退出当前Activity时，当前Activity从中弹出，前一个Activity恢复执行。当所有Activity均从栈中弹出后，没有任务，栈会被回收到

###### 启动模式

启动模式是指Activity的新实例如何与当前任务栈关联，以什么方式进入到任务栈中

- 声明方式：

  1. 在AndroidManifest.xml中指定启动模式，当前Activity在任何情况下都会执行该启动模式
  2. 使用Intent标志定义，可以控制Activity何时启动时才应用启动模式

  > 如果一个Activity两种方式都声明了，使用Intent标志的优先级要比AndroidManifest的高，且两种方式的启动模式有些不一样，Intent标志中定义的有些Maniest中没有。所以通常所说的四种启动模式指的是AndroidManifest中声明的方式

- AndroidManifest中声明启动模式(四种启动模式)

  - **standard**:标准模式

    默认启动模式，直接创建新的实例压入启动它的任务栈顶

  - **singleTop**:栈顶复用模式

    启动`singleTop`模式的Activity时，如果发现当前任务的栈顶已存在该Activity实例，就不会创建新的实例，而是调用该实例的`onNewIntent`方法，其他与标准模式一样

  - **singleTask**:栈内复用模式

    当启动该模式的Activity时，系统会判断当前是否有它想要的任务栈，如果没有系统会创建一个新的任务，并将该Activity实例作为该任务的根Activity;如果有且任务栈里只有它自己，那么会直接调用该实例的`onNewIntent`方法；如果任务栈中还有其它实例，会将该Activity上方的其它Activity全部出栈

    > taskAffinity:任务相关性，如果你在`<activity>`标签没指定这个属性，那么它就用`<application>`标签的`taskAffinity`属性，如果`<application>`标签下也没指定，它就应用包名当做默认值。

  - **singleInstance**:单例模式

    为Activity单独创建一个任务并能够复用

- 使用Intent声明启动模式

  - 方法

    ```java
    intent.addFlags(int flags);
    intent.setFlags(int flags);
    ```

  - 典型参数

    - **Intent.FLAG_ACTIVITY_SINGLE_TOP**

      与`singleTop`启动模式一致

    - **Intent.FLAG_ACTIVITY_NEW_TASK**

      与`singleTask`启动模式一致

    - **Intent.FLAG_ACTIVITY_CLEAR_TOP**

      如果将启动的Activity已存在于当前任务栈中，则会弹出销毁它上方的所有Activity,并调用该Activity实例的`onNewIntent`方法，而不是启动该Activity实例

##### 四. 启动流程

见[Android开发艺术探索第九章#Activity](../../Android开发艺术探索/Android开发艺术探索第九章笔记.md#Activity的工作过程)

##### 五. 数据通信

​		Activity之间通过Intent传递数据，Intent传递数据大小是有限制的，由于Intent底层使用了`binder`机制，所以这其实是Binder传递数据的大小。其机制中使用了一个共享内存-`Binder transaction buffer`其大小限制为`1MB`，是公用的。所以不管是一次传递大数据还是同时传递多个数据，都不能超过`1MB`

##### 六. scheme跳转协议

- 定义

  scheme是一种页面跳转协议，通过其可以跳转到App的各个页面。可以通过scheme跳转到另一个app的页面，也可以通过H5页面跳转原生App页面

- 格式

  ```
  协议名称 : // 作用地址域 : 路径端口 / 指定页面路径 ？ 传递参数
  ```

- 使用

  1. 在Manifest中定义一个Scheme,进行配置
  2. 在代码中获取Scheme跳转参数
  3. 通过Intent或Html进行调用

##### 七. Activity的管理机制

![](https://upload-images.jianshu.io/upload_images/7508328-f8e1532b18ad6317.png?imageMogr2/auto-orient/strip|imageView2/2/w/494/format/webp)
图片来自网络

- AMS提供了一个mHistory来管理所有的Activity,Activity在AMS中以ActivityRecord形式存在，所有的ActivityRecord会被存储在mHistory中管理
- Task在AMS中以TaskRecord形式存在，每个ActivityRecord对应一个TaskRecord,有相同TaskRecord的ActivityRecord在mHistory中处于连续的位置
- 进程在AMS中的管理形式为ProcessRecord,同一个TaskRecord的Activity可能处于不同的进程中,每个Activity所处的进程和Task没有关系
- Activity启动时ActivityManagerService会为其生成对应的ActivityRecord记录，并将其加入到回退栈(back stack)中，另外也会将ActivityRecord记录加入到某个Task中。

##### 八. 进程

​		共有`前台进程`，`可见进程`，`服务进程`，`后台进程`，`空进程`五种，优先级依次降低

- 前台进程

  用户正在交互的Activity;主动调用的前台服务

- 可见进程

  处于*onPause*没有进入*onStop*的Activity;绑定到前台Activity的Service

- 服务进程

  普通的startService

- 后台进程

  处于*onStop*的Activity

- 空进程

  没有任何活动的组件的进程，出于缓存的目的