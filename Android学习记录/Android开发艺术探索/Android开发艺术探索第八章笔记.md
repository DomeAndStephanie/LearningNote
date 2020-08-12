### 理解Window和WindowManager

---

**Window**表示一个窗口的概念，是一个抽象类，它的具体实现是`PhoneWindow`。创建一个**Window**需要通过**WindowManage**来完成，**WindowManager**是外界访问**Window**的入口，**Window**的具体实现在`WindowManagerService`中，**WindowManager**和`WindwManagerService`的交互是一个**IPC**过程。Android中所有的视图都是通过**Window**来呈现的，**Window**是**View**的直接管理者。

#### 1. Window和WindowManager

​	WindowManager常用的有三个方法：添加View, 更新View, 删除View

```java
//添加View
mButton = new Button(this);
mLayoutParams = new WindowManager.LayoutParams(....);
mLayoutParams.flags = LayoutParams.FLAG_NOT_TOUCH_MODAL...;
mLayoutParams.gravity = Gravity.LEFT;
mLayoutParams.x = 100;
mLayoutParams.y = 300;
mWindowManager.addView(mButton, mLayoutParams);
```

> - `flags`参数表示Window的属性：
>
>   FLAG_NOT_FOCUSABLE:Window不需要获取焦点，也不需要接收各种输入事件
>
>   FLAG_NOT_TOUCH_MODAL: 系统会将当前Window区域以外的单击事件传递给底层的Window,区域内View处理
>   FLAG_SHOW_WHEN_LOCKED: 可以让Window显示在锁屏的界面上
>
> - Window是分层的，每个Window都有对应的`z-ordered`，层级大的会覆盖在层级小的上面。
>
> - `type`参数表示Window的类型，Window有三种类型，分别为
>
>   `应用Window` : 对应着一个Activity.层级范围是==1~99==
>
>   `子Window` : 不能单独存在，需要附属在特定的父Window中，比如dialog等，层级范围是==1000~1999==
>
>   `系统Window` ：需要声明权限才能创建的Window，比如Toast和系统状态栏，层级范围是==2000~2999==

#### 2. Window的内部机制

​	每个Window都对应着一个View和一个ViewRootImpl,Window和View通过ViewRootImpl来建立联系。

1. Window的添加过程

   ```mermaid
   sequenceDiagram
   Actor ->>+ Window : addView
   Window ->>+ WindowManager : addView
   WindowManager ->>+ WindowManagerImpl : addView
   WindowManagerImpl ->>+ WindowManagerGlobal : addView
   WindowManagerGlobal ->> WindowManagerGlobal : 检查参数是否合法
   WindowManagerGlobal ->> WindowManagerGlobal : 创建ViewRootImpl,添加View列表
   WindowManagerGlobal ->> ViewRootImpl : 更新界面
   ViewRootImpl ->> WindowSession : 添加Window
   WindowSession ->> WindowManagerService : 添加Window
   ```

2. Window的删除过程

   ```mermaid
   sequenceDiagram
   Window... ->> WindowManagerImpl :removeView
   WindowManagerImpl ->> WindowManagerImpl : 查找待删除的View索引
   WindowManagerImpl ->> WindowManagerImpl : 调用removeViewLocked完成删除
   ```

   > 真正删除的逻辑在`dispatchDetachedFromWindow`方法中实现
   >
   > 1. 垃圾回收
   > 2. 通过Session的remove方法删除Window
   > 3. 调用View的dispatchDetachedFromWindow方法
   > 4. 调用WindowManagerGlobal的doRemoveView方法刷新数据

3. Window的更新过程

   更新逻辑在**WindowManagerGlobal**的`updateViewLayout`中

   1. 更新View的LayoutParams并替换掉原有的
   2. 更新ViewRootImpl中的LayoutParams

#### 3. Window的创建过程

1. Activity的Window创建过程

   ```mermaid
   sequenceDiagram
   Actor ->> ActivityThread : startActivity
   ActivityThread ->> ActivityThread : performLaunchActivity
   ActivityThread ->> PolicyManager : attach
   PolicyManager ->> PolicyManager : makeNewWindow
   PolicyManager ->> Policy : makeNewWindow
   ```

   Activity将具体实现交给了Window，所以其逻辑在`PhoneWindow`中,大致步骤如下：

   1. 如果没有DecorView，就创建它
   2. 将View添加到DecorView的mContentParent中
   3. 回调Activity的onContentChanged方法通知视图改变
   4. 调用Activity的makeVisible方法，完成DecorView的添加和显示

2. Dialog的Window创建过程

   1. 创建Window
   2. 初始化DecorView并将Dialog的视图添加到DecorView中
   3. 将DecorView添加到Window中并显示

   > 普通的Dialog必须采用Activity的Context，否则会报缺失token的错误，应用token一般只有Activity拥有，但系统Window可以不需要token

3. Toast的Window创建过程

   在Toast内部有两类IPC过程，第一类是Toast访问`NotificationManagerService`,第二类是`NotificationManagerService`回调Toast中的TN接口。Toast属于系统Window。

   ```mermaid
   sequenceDiagram
   Toast ->> NotificationManagerService : show
   NotificationManagerService ->> NotificationManagerService : enqueueToast
   NotificationManagerService ->> NotificationManagerService : showNextToastoLocked
   NotificationManagerService ->>NotificationManagerService : scheduleTimeoutLocked
   ```

   