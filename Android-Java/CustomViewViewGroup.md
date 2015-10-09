# 自定义View/ViewGroup以高性能实现自定义UI

## [View绘制流程](http://a.codekk.com/detail/Android/lightSky/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20View%20%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B)
当 Activity 接收到焦点的时候，它会被请求绘制布局，该请求由 Android framework 处理。绘制是从根节点开始，对布局树进行 measure 和 draw。整个 View 树的绘图流程在ViewRoot.java类的performTraversals()函数展开，该函数所做的工作可简单概况为是否需要重新计算视图大小(measure)、是否需要重新安置视图的位置(layout)、以及是否需要重绘(draw)。

+  measure
  +  ViewGroup负责测量所有子View及自己，View负责测量自己；
  +  View重写onMeasure以实现自己的测量逻辑；
  +  View在onMeasure中，根据传入的widthMeasureSpec, heightMeasureSpec（父ViewGroup对自己的限制），以及自己的内容显示需要，计算出想要的宽高，通过setMeasuredDimension进行最终设置（该方法必须调用，否则抛异常），设置的宽高也是经过位运算的值；
  +  MeasureSpec值都是经过位运算的，View类提供了一些方法从中读取信息
  +  推荐按照以下方式实现
  
    ```java
      @Override
      protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
  
          // Try for a width based on our minimum
          int minw = getPaddingLeft() + getPaddingRight() + getSuggestedMinimumWidth();
          int w = resolveSizeAndState(minw, widthMeasureSpec, 1);
  
          // Whatever the width ends up being, ask for a height that would let the pie
          // get as big as it can
          int minh = getPaddingBottom() + getPaddingTop() + getSuggestedMinimumHeight();
          int h = resolveSizeAndState(minh, heightMeasureSpec, 0);
  
          setMeasuredDimension(w, h);
  
      }
    ```
  +  计算自己所需高度、宽度则通过重写getSuggestedMinimumWidth，getSuggestedMinimumHeight来进行计算
  +  子View measure完之后，如果不符合父ViewGroup给出的约束（过大或过小），将触发父ViewGroup对子View进行第二次measure，此时传入的约束可能会发生变化，例如从限制最大值/最小值到给定具体值。
  
  +  ViewGroup的measure过程包括：对于每个child，根据自己的measureSpec、child的LayoutParam、自己的padding，来计算child的measureSpec，然后传给child，让其自己进行上述的measure逻辑
  
+  layout
  +  首先要明确的是，子视图的具体位置都是相对于父视图而言的。View 的 onLayout 方法为空实现，而 ViewGroup 的 onLayout 为 abstract 的，因此，如果自定义的 View 要继承 ViewGroup 时，必须实现 onLayout 函数。
  +  measure完成后，就是根据每个子View确定显示的大小，及其显示规则，来排布每个子View了，这个排布就是设置子View的left, top, right, bottom了，注意子View的这些值都是相对于父ViewGroup的，而不是屏幕坐标系的绝对位置。
  
+  draw
  +  应该重写的是onDraw方法，一定要重写draw方法时一定要首先调用`super.draw()`

+  一定要注意
  +  自定义View/ViewGroup一定要避免调用requestLayout，会导致整个view hierarchy的重绘，影响性能
  
  
## [自定义View实例](https://medium.com/android-news/prefmatters-using-custom-views-in-android-to-improve-performance-part-1-4dc9bdd75396)

## [自定义ViewGroup实例](https://medium.com/android-news/perfmatters-introduction-to-custom-viewgroups-to-improve-performance-part-2-f14fbcd47c)