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

