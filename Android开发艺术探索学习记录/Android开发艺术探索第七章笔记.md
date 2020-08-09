### Android动画深入分析

---

#### 1. View动画

​	View动画的作用对象是View,支持`平移动画`,`缩放动画`,`旋转动画`,`透明度动画`四种动画效果

|    名称    |    标签     |       子类        |      效果      |
| :--------: | :---------: | :---------------: | :------------: |
|  平移动画  | <translate> | TransateAnimation |    移动View    |
|  缩放动画  |   <scale>   |  ScaleAnimation   | 放大或缩小View |
|  旋转动画  |  <rotate>   |  RotateAnimation  |    旋转View    |
| 透明度动画 |   <alpha>   |  AlphaAnimation   | 改变View透明度 |

##### 	1.使用步骤

1. 在res/anim/路径下创建动画的xml文件

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <set xmlns:android="http://scemas.android.com/apk/res/android"
        android:fillAfter="true"
        android:zAdjustment="normal" >
   
       <rotate
               android:duration="400"
               android:fromDegress="0"
               android:toDegress="90" />
   </set>
   ```

2. 在代码中调用

   ```java
   Animation animation = AnimationUtils.loadAnimation(this, R.anim.animation_file);
   mButton.startAnimation(animation)
   ```

   > 也可直接在代码中进行设置和调用
   >
   > ```java
   > AlphaAnimation alphaAnim = new AlphaAnimation(0, 1);
   > alphaAnim.setDuration(300);
   > mButton.startAnimation(animation)
   > ```

##### 	2.自定义View动画

   1. 继承Animation
   2. 重写`initialize`和`applyTransformation`方法

   > initialize : 初始化
   >
   > applyTransformation : 进行相应的矩阵变换

#### 2. 帧动画

​	帧动画是顺序播放一组预先定义好的图片。

#####     1. 使用步骤

 1. 通过XML定义AnimationDrawable

    ```xml
    <!-- res/drawable/frame_anim.xml -->
    <?xml version="1.0" encoding="utf-8"?>
    <animation-list xmlns:android="http://scemas.android.com/apk/res/android"
                    android:oneshot="false">
        <item andrid:drawable="@drawable/image1" android:duration="500" />
        <item andrid:drawable="@drawable/image2" android:duration="500" />
        <item andrid:drawable="@drawable/image3" android:duration="500" />
    </animation-list>
    ```

	2. 代码中调用

    ```java
    mButton.setBackroundResource(R.drawable.frame_anim);
    AnimationDrawable drawable = (AnimationDrawable) mButton.getBackground();
    drawable.start();
    ```

#### 3. View动画的特殊使用场景

1. LayoutAnimation

   LayoutAnimation作用于ViewGroup,为ViewGroup指定一个动画，子元素都会具有该动画效果

2. Activity的切换效果

   主要用到`overridePendingTransition(int enterAnim, int exitAnim)`方法，这个方法必须在**startActivity**或者**finish**方法之后调用才能生效

   enterAnim ： Activity被打开时的动画资源

   exitAnim ： Activity被暂停时的动画资源

#### 4. 属性动画

​		属性动画可以对任意对象的属性进行动画而不仅仅是View,动画的默认时间间隔是300ms,默认帧率是10ms/帧，其可以达到的效果是：在一个时间间隔内完成对象从一个属性值到另一个属性值的改变。

> 插值器(TimeInterpolator)：根据时间流逝的百分比来计算当前属性值改变的百分比
>
> 估值器(TypeEvaluator)：根据当前属性改变的百分比来计算改变后的属性值

属性动画的监听器：

```java
public static interface AnimatorListener{
    void onAnimationStart(Animator animation);
    void onAnimationEnd(Animator animation);
    void onAnimationCancel(Animator animation);
    void onAnimationRepeat(Animator animation);
}
```

属性动画的工作原理：





