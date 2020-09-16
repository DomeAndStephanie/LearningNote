[TOC]

#### Android面试题总结

---

##### 四大组件是什么

- Activity: 展示型组件，向用户展示界面和提供用户界面交互
- Service: 在后台执行长时间运行操作而没有用户界面的应用组件
- BroadcastReceiver:消息型组件，在不同组件和应用之间传递消息
- ContentProvider:数据共享组件，向其他组件或应用共享数据

##### 四大组件的生命周期和用法

- Activity

  Activity的四种状态

  > active:运行状态，当前Activity获取焦点
  >
  > paused:暂停状态，当前Activity失去焦点，仍然可见
  >
  > stopped:不可见但成员变量等依然存在
  >
  > killed:被销毁，成员变量等被收回

  正常情况下的声明周期：

  > Activity启动：onCreate->onStart->onResume
  >
  > 点击Home键：onPause->onStop
  >
  > 返回原Activity：onRestart->onStart->onResume
  >
  > 退出当前Activity：onPause->onStop->onDestroy

  Dialog弹出时：

  > 1. 创建的dialog由于依附的Activity没有被遮挡，不会走生命周期方法
  > 2. 跳转到一个不是全屏的Activity,onPause->onStop

  横竖屏切换时：

  > 1. 不设置`android:configChanges`，切屏会重新调用各个生命周期，切横屏时执行一次，切竖屏时执行两次
  > 2. 设置`android:configChanges=“orientation"`时，切屏还是会重新调用各个生命周期，横屏和竖屏只会执行一次
  > 3. 设置`android:configChanges="orientation|keyboardHidden"`时，切屏不会调用各个生命周期，只会执行`onConfigurationChanged`方法

  锁屏与解锁屏幕

  锁屏时只会调用onPause,不会调用onStop,解锁后则调用onResume

- BroadcastReceiver

  

- Service

- ContentProvider

  

  

  

  

