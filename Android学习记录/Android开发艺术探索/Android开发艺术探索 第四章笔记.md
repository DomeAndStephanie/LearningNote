### Android开发艺术探索 第四章笔记

#### 第四章  View的工作原理

1. 初识ViewRoot和DecorView

   ViewRoot:对应于ViewRootImpl类，是连接WindowManager和DecorView的纽带，View的三大流程(measure,layout,draw)来完成的。

   ```java
   //在ActivityThread中，当Activity对象被创建完后，会将DecorView添加到Window中，同时创建 
   //ViewRootImpl对象，并将这两者关联起来
   root = new ViewRootImpl(view.getContext(),display);
   root.setView(view,wparams,panelParentView);
   ```

   View的绘制流程从ViewRoot的performTraversals方法开始，经过三大流程完成绘制。

   ![图4-1performTraversals的工作流程图](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/图4-1performTraversals的工作流程图.png)

   ​                                            图 4-1 performTraversals的工作流程图

   > measure:测量View的宽和高
   >
   > layout:确定View在父容器中的放置位置
   >
   > draw:将View绘制在屏幕上

2. 理解MeasureSpec

   在很大程度上决定了一个View的尺寸规格(会受到父容器的影响)。

   1. MeasureSpec:代表一个32位int值，高2位代表**SpecMode**，低30位代表**SpecSize**。

      - SpecMode：测量模式
      - SpecSize：在某种测量模式下的规格大小

      ```java
      private static final int MODE_SHIFT = 30;
      private static final int MODE_MASK = 0x3 << MODE_SHIFT;
      private static final int UNSPECIFIED = 0 << MODE_SHIFT;
      private static final int EXACTLY = 1 << MODE_SHIFT;
      private static final int AT_MOST = 2 << MODE_SHIFT;

      public static int makeMeasureSpec(int size,int mode){
        if(sUseBrokenMakeMeasureSpec){
          return size + mode;
        }else{
          return (size & ~MODE_MASK) | (mode & MODE_MASK)
        }
      }
      public static int getMode(int measureSpec){
        return (measureSpec & MODE_MASK);
      }
      public static int getSize(int measureSpec){
        return (measureSpec & ~MODE_MASK);
      }
      ```

      - **UNSPECIFIED**:父容器不对View有任何限制
      - **EXACTLY**：父容器已检测出View所需要的精确大小，对应于LayoutParams中的match_parent和具体的尺寸数值。
      - **AT_MOST**：父容器给View指定了可用大小，View的大小不能大于这个值，对应于LayoutParams中的wrap_content。

   2. MeasureSpec和LayoutParams的对应关系

      1. 顶级View（Decoriew）：Measurespec由窗口的尺寸和其自身的LayoutParams来共同确定；

         - MATVH_PARENT：精确模式，大小就是窗口的大小；
         - WRAP_CONTENT:最大模式，大小不定，不能超过窗口的大小；
         - 固定大小：精确模式，大小为LayoutParams中指定到大小。
         
      2. 普通View:Measurespec由父容器的MeasureSpec和自身到LayoutParams共同决定。
      
         TODO
      
         ![](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/普通View的MeasureSpec的流程.png)

3. View的工作流程

   1. measure过程
   
      - View的measure过程
   
        ```java
        //在View的measur方法中会去调用View的onMeasure方法
        protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
            setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(),widthMeasureSpec),getDefaultSize(getSuggestedMinimumHeight(),heightMeasureSpec));
        }
        ```
   
        > getSuggestedMinimumXXX:如果View没有设置背景，那么返回`android:minXXX`这个属性的值；如果View设置了背景，则返回`android:minXXX`和背景的最小尺寸这两个的最大值。
        >
        > 直接继承View的自定义控件需要重写onMeasure方法并设置wrap_content时的大小，否则在布局中使用wrap_content就相当于使用match_parent
   
      - ViewGroup的measure过程
   
        除了完成自己的measure过程外，还需要遍历所有子元素的measure方法。
   
        ```java
        //ViewGroup是抽象类，没有重写onMeasure方法，提供了measureChildren方法
        protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
            final int size = mChildrenCount;
            final View[] children = mChildren;
            for(int i = 0; i < size; ++i) {
                final View child = children[i];
                if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                    measureChild(child, widthMeasureSpec, heightMeasureSpec);
                }
            }
        }
        ```
   
        > measure完成后，通过`getMeasuredWidth/Height`方法就可以正确获取到View的宽高，但由于存在多次measure的情况，所以在onMeasure中得到的测量宽高不一定准确，**在onLayout方法中获取宽高比较好一点**。
        >
        > View的measue过程和Activity的生命周期方法不是同步执行的。可以通过以下方法解决：
        >
        > - Activity/View#onWindowFocusChanged
        > - view.post(runnable)
        > - ViewTreeObserver
        > - view.measure(int widthMeasureSpec, int heightMeasureSpec)
      
   2. layout过程
   
      Layout的作用是ViewGroup用来确定子元素的位置，当ViewGroup的位置被确定后，它在onLayout中会遍历所有的子元素并调用其layout方法，在layout方法中onLayout又会被调用。
   
      - View的layout方法流程
   
        首先通过`setFrame`方法来设定View的四个顶点的位置，接着调用onLayout方法。
   
      > 在View的默认实现中，View的测量宽高和最终宽高是相等的，不同点是测量宽高形成于View的measure过程，最终宽高形成于View的layout过程。
   
   3. draw过程
   
      将View绘制到屏幕上，绘制过程如下：
   
      1. 绘制背景：backgroud.draw(canvas)
      2. 绘制自身：onDraw
      3. 绘制子元素：dispatchDraw
      4. 绘制装饰：onDrawScrollBars
   
      View绘制过程的传递通过dispatchDraw实现，该方法会遍历调用所有子元素的draw方法
   
4. 自定义View

   1. 自定义View的分类

      ```mermaid
      graph LR
      A[自定义View] --> B[继承View重写onDraw方法]
      A --> C[继承ViewGroup派生特殊的Layout]
      A --> D[继承特定的View 如TextView等]
      A --> E[继承特定的ViewGroup 如LinearLayout等]
      ```

   2. 自定义View须知

      - 让View支持`wrap_content`
      - 支持`Padding`
      - 尽量不要在View中使用Handler
      - View中如果有线程或者动画，需要及时停止(`View#onDetachedFromWindow`)
      - 处理好滑动冲突
