### Android开发艺术探索 第三章笔记

#### 第三章 View的事件体系

##### 3.1 View基础知识

1. 什么是View

   View是Android中所有控件的基类，是一种界面层的控件的一种抽象。ViewGroup内部包含了许多控件，是一组View，它们之间的关系如下图。

   ![图3-1View的层级结构](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图3-1View的层级结构.png)

2. View的位置参数

   View的位置由它的四个顶点决定，分别对应于View的四个属性：**top, left, right, bottom **即它的四个坐标，并且这四个坐标都是相对与View的父容器来说的，是相对坐标。

   ![图3-2View的位置坐标和父容器的关系](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图3-2View的位置坐标和父容器的关系.png)

   > width = right - left
   >
   > height = bottom - top

3. MotionEvent和TouchSlop

   1. MotionEvent:在手指接触屏幕后所产生的一系列事件

      - ACTION_DOWN——手指刚接触屏幕
      - ACTION_MOVE——手指在屏幕上移动
      - ACTION_UP——手指从屏幕上松开的一瞬间

      1. 典型事件序列

      - 点击屏幕后离开松开，事件序列为——DOWN -> UP;
      - 点击屏幕滑动一会儿在松开，事件序列为——DOWN->MOVE->...->MOVE->UP

      2. 通过MotionEvent对象可以得到点击事件发生的**x**和**y**坐标。

      - getX/getY——返回的是相对于当前View左上角的x和y坐标；
      - getRawX/getRawY——返回的是相对于手机屏幕左上角的x和y坐标。

   2. TouchSlop：系统所能识别的滑动的最小距离，当两次滑动之间的距离小于这个常量时，系统不能识别为是滑动事件，值和设备有关，通过如下方法可以获取这个常量。

      ```java
      ViewConfiguration.get(getContext()).getScaledTouchSlop();
      ```

4. VelocityTracker,GestureDetector和Scroller

   1. VelocityTracker：速度追踪，用于追踪手指在滑动过程中的速度，包括水平和竖直方向。

      ```java
      //1.首先在View的onTouchEvent方法中追踪当前点击事件的速度
      VelocityTracker velocityTracker = VelocityTracker.obtain();
      velocityTracker.addMovement(event);
      //2.获取当前速度
      velocityTracker.computeCurrentVelocity(1000);//计算一个时间段内的速度
      int xVelocity = (int) velocityTracker.getXVelocity();
      int yVelocity = (int) velocityTracker.getYVelocity();
      //3.重置并回收内存
      velocityTracker.clear();
      velocityTracker.recycle();
      ```

   2. GestureDetector：手势检测，用于辅助检测用户的单击，滑动，长按，双击等行为。

      ```java
      //1.创建一个GestureDetector对象，实现onGestureListener接口
      GestureDetector mGestureDetector = new GestureDetector(this);
      mGestureDetector.setIsLongpressEnabled(false);//解决长按屏幕后无法拖动的现象
      //2.接管目标View的onTouchEvent方法,在其中使用
      boolean consume = mGestureDetector.onTouchEvent(event);
      return consume;
      //3.选择实现onGestureListener和OnDoubleTapListener中的方法
      //比如onSingleTapup():单击 ，onFling():滑动，onScroll():拖动，onLongPress():长按，
      //onDoubleTap():双击
      ```

   3. Scroller：弹性滑动对象，用于实现View的弹性滑动。

      ```java
      Scroller mScroller = new Scroller(mContext);
      //缓慢滚动到指定位置
      private void smoothScrollTo(int destX,int destY){
        int scrollX = getScrollX();
        int delta = destX-scrollX;
        mScroller.startScroll(ScrollX,0,delta,0,1000);
      }
      @Override
      public void computeScroll(){
        if(mScroller.computeScrollOffset()){
          smoothScrollTo(mScroller.getCurrX(),mScroller.getCurrY());
          postInvalidate();
        }
      }
      ```

##### 3.2 View的滑动

1. 使用ScrollTo/scrollBy

   ```java
   public void scrollTo（ int x,int y）{
     if(mScrollX != x||mScrollY !=y){
       int oldX = mScrollX;
       int oldY = mScrollY;
       mScrollX = x;
       mScrollY = y;
       invalidateParentCaches();
       onScrollChanged(mScrollX,mScrollY,oldX,oldY);
       if(!awakenScrollBars()){
         postInvalidateOnAnimation();
       }
     }
     public void scrollBy(int x,int y){
       scrollTo(mScrollX + x,mScrollY + y);
     }
   }
   ```

   > 在滑动过程中，mScrollX的值总是等于View左边缘和View内容左边缘在水平方向的距离；mScrollY的值等于View上边缘和View内容上边缘在竖直方向上的距离
   >
   > scrollTo和scrollBy只能改变View内容的位置而不能改变View在布局中的位置。
   >
   > 如果从左向右滑动，那么mScrollX为负值，反之为正值；如果从上往下滑动，那么mScrollY为负值，反之为正值。

   ![图3-3mScrollX和mScrollY的变换规律](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图3-3mScrollX和mScrollY的变换规律.png) b

2. 使用动画

   ```xml
   <!--View动画实现移动-->
   <translate
              android:duration="100"
              android:fromXDelta="0"
              android:fromYDelta="0"
              android:interpolator="@android:anim/linear_interpolator"
              android:toXDelta="100"
              android:toYDelta="100"/>
   ```

   ```java
   //属性动画实现移动
   ObjectAnimator.ofFloat(targetView,"translationX",0,100)
                 .setDuration(100)
                 .start();
   ```

3. 改变布局参数：即改变**LayoutParams**

   ```java
   MarginLayoutParams params = (MarginLayoutParams)mButton.getLayoutParams();
   params.width +=100;
   params.leftMargin +=100;
   mButton.requestLayout(); //或者 mButton.setLayoutParams(params);
   ```

4. 各种滑动方式的对比

   - scrollTo/scrollBy :  操作简单，适合对View内容的滑动
   - 动画 ：操作简单，主要适用于没有交互的View和实现复杂的动画效果
   - 改变布局参数 ：操作稍复杂，适用于有交互的View

##### 3.3 弹性滑动

1. 使用Scroller

   当View重绘后会在在`draw`方法中调用**computeScroll**，而computeScroll又会去向当前**Scroller**获取当前的`scrollX`和`scrollY`；然后通过`scrollTo`方法实现滑动，然后继续重绘，循环反复。

2. 通过动画

3. 使用延时策略

   通过发送一系列延时消息达到渐进式的效果，可以使用`Handler`或View的`postDelayed`方法。

##### 3.4 View的事件分发机制

1. 点击事件的传递规则

   ```java
   //点击事件分发过程中的三个重要方法
   public boolean dispatchTouchEvent(MotionEvent e)  //用来进行事件的分发
   public boolean onInterceptTouchEvent(MotionEvent e) //用来判断是否拦截某个事件
   public boolean onTouchEvent(MotionEvent e)  //在dispatchTouchEvent中调用，用来处理点击事件
   //三个方法的关系（伪代码）
   public boolean dispatchTouchEvent(MotionEvent e){
     boolean consume =false;
     if(onInterceptTouchEvent(e)){
       consume =onTOuchEvent(e);
     }else{
       consume = child.dispatchTouchEvent(e);
     }
     return consume;
   }
   ```

   当一个点击事件产生时，它的传递过程为：**Activity -> Window ->View**,即事件先传递给Activity，Activity再传递给Window，最后Window再传递给顶级View，顶级View接收到事件后，就会按照事件分发机制去分发事件。

   > PS：
   >
   > 1. 同一个事件序列是指从手指接触屏幕那一刻起到手指离开屏幕的那一刻结束，在这一过程中产生的一系列事件；
   > 2. 正常情况下，一个事件序列只能被一个View拦截且消耗；
   > 3. 某个View一旦决定拦截，那么这个事件序列都只能由它处理，并且它的onInterceptTouchEvent不会被调用；
   > 4. 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN(onTouchEvent返回false)，那么同一事件序列的其它事件都不会交由它处理，事件会重新交由它的父元素处理(调用父元素的onTouchEvent);
   > 5. ViewGroup默认不拦截任何事件；
   > 6. View没有onInterceptTouchEvent方法，一旦有点击 事件传递给它，它的onTouchEvent方法就会调用；
   > 7. View的onTouchEvent默认都会消耗事件(返回true)。除非它是不可点击的(clickable和longClickable都为false)；
   > 8. View的enable属性不影响onTouchEvent的默认返回值。、
   > 9. onClick会发生的前提是当前View是可点击的，并且它收到了down和up事件；
   > 10. 事件传递过程是由外向内的，即事件总是先传递给父元素，再由父元素分发给子View，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外。

2. 事件分发的源码解析

   1. Activity对点击事件的分发过程

      当一个点击操作发生时，事件最先传递给当前Activity，由Activity的dispatchTouchEvent来进行事件分发，具体是由Activity的Window来完成的。Window会将事件传递给**decor view**，decor view 一般是当前界面的底层容器(即setContentView所设置的View的父容器)。

      ```java
      //Activity # dispatchTouchEvent
      public boolean dispatchTouchEvent(MotionEvent ev){
        if(ev.getAction() == MotionEvent.ACTION_DOWN){
          onUserInteraction();
        }
        //交由所附属的Window进行分发
        if(getWindow().superDispatchTouchEvent(ev)){
          return true;    
        }
        return onTouchEvent(ev);//所有View都没有处理，调用Activity的onTouchEvent()
      }
      ```

      ```java
      //Window的分发
      //Window # superDispatchTouchEvent
      public abstract boolean superDispatchTouchEvent(MotionEvent event)
      //Window的唯一实现是PhoneWindow
      //PhoneWindow # superDispatchTouchEvent
      public boolean superDispatchTouchEvent(MotionEvent event){
        return mDecor.superDispatchTouchEvent(event);
      }
      //DecorView
      private final class DecorView extends FrameLayout implements RootViewSurfaceTaker{
        private DecorView mDecor;
        @Override
        public final View getDecorView(){
          if(mDecor == null){
            installDecor();
          }
          return mDecor;
        }
      }
      ```

   2. 顶级View对点击事件的分发过程

      点击事件到达顶级View(一般是ViewGroup)以后，会调用ViewGroup的**dispatchTouchEvent**方法，然后如果ViewGroup拦截事件(**onInterceptTouchEvent**返回true)，则事件由ViewGroup处理，此时如果ViewGroup设置了**onTouchListener**，则**onTouch**会被调用，否则调用**onTouchEvent**，且如果设置了**onClickListener**，则会调用**onClick**。如果顶级View不拦截，则事件会传递给它所在的点击事件链上的子View，调用子View的**dispatchTouchEvent**，如此循环，完成整个事件的分发。

      ![图3-4顶级View的事件分发](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图3-4顶级View的事件分发.png)

      ```java
      //check for interception # dispatchTouchEvent # ViewGroup
      final boolean intercepted;
      if(actionMasked == MotionEvent.ACTION_DOWN || mFirstTouchTarget != null){
        final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
        if(!disallowIntercept){
          intercepted = onInterceptTouchEvent(ev);
          ev.setAction(action);//restore action in case it was changed
        }else{
          intercepted = true;
        }
      }else{
        intercepated = true;
      }
      ```

      > 1. `onInterceptTouchEvent`不是每次事件都会被调用，当事件能够传递到当前的**ViewGroup**时，`dispatchTouchEvent`会每次都调用
      > 2. `FLAG_DISALLOW_INTERCEPT`标记可以让**ViewGroup**不再拦截事件
      
   3. View对点击事件的处理过程
   
      ![](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/View点击事件处理流程.png)


##### 3.5 View的滑动冲突

1. 常见的滑动冲突场景

   1. 外部滑动方向和内部滑动方向不一致

      ![图3-5常见滑动冲突场景1](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图3-5常见滑动冲突场景1.png)

   2. 外部滑动方向和内部滑动方向一致

      ![图3-6常见滑动冲突场景2](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图3-6常见滑动冲突场景2.png)

   3. 上面两种情况的嵌套

      ![图3-7常见滑动冲突场景3](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图3-7常见滑动冲突场景3.png)

2. 滑动冲突的处理规则

   - 第一种滑动冲突：根据滑动时水平滑动还是竖直滑动来判断由谁拦截事件。
   - 第二种滑动冲突：根据实际业务需求来进行处理
   - 第三种滑动冲突：与第二种一致

3. 滑动冲突的解决方式

   1. 外部拦截法

      点击事件都先经过父容器的拦截处理，根据父容器的需求即可解决滑动冲突

   2. 内部拦截法

      父容器不拦截任何事件，所有的事件都传递给子元素，子元素可以直接消耗掉或者交由父容器进行处理。需要配合`requestDisallowInterceptTouchEvent`方法才能正常工作。