## [Canvas & Drawables](http://developer.android.com/guide/topics/graphics/2d-graphics.html)
+  绘制4个基本元素
  +  Bitmap：保存每个像素的数据
  +  Canvas：提供draw*** API，通过draw系列函数（绘制线、矩形、圆、椭圆等），把绘制结果写入到bitmap对象中
  +  Drawing primitive：Rect, Path, text, Bitmap...
  +  Paint：画笔，描述绘制内容的属性（颜色、样式）
+  两种用法
  +  绘制到View，只需要在View子类的onDraw方法中绘制即可，onDraw函数会传入一个Canvas对象，使用其进行绘制即可，由framework负责绘制流程；适用于静态自定义图形、简单动态图形；
  +  绘制到Canvas，绘制原语一样，但是最后需要显示的时候，要通过View/Surface来进行显示；适合复杂的动态图形绘制，例如视频游戏；
+  绘制到View
  +  适用于静态、低帧率、简单动态图形的绘制，重写View的onDraw方法即可
  +  使用参数传入的Canvas对象，由framework负责调用onDraw函数
  +  通过invalidate函数请求重绘
  +  onDraw、invalidate都需要在主线程执行，其他线程可以通过postInvalidate请求重绘
  +  
+  绘制到Canvas
  +  Canvas记录（执行）draw操作，将操作记录到Bitmap上，最后将Bitmap显示在Surface上
+  绘制到SurfaceView
  +  SurfaceView是View的子类，支持他线程绘制，主线程不同步等待其绘制，不需要保证60 fps
+  Drawable
  +  2D图形的高度抽象，有一系列的子类
  
## [Drawable Resources](http://developer.android.com/intl/zh-cn/guide/topics/resources/drawable-resource.html)
+  BitmapDrawable，用图片（.png, .jpg, or .gif）创建drawable，xml定义为：
    ```xml
    <bitmap
            android:src="@mipmap/ic_launcher"
            android:gravity="top"
            />
    ```
+  NinePatchDrawable，点9图（.9.png），可以在两个方向上拉伸而不会变形
+  LayerDrawable，通过xml定义`<layer-list>`，里面定义多个`<item>`来定义多层的drawable
+  StateListDrawable，通过xml定义`<selector>`，里面定义多个`<item>`来定义具有不同状态的drawable，实现按钮点击态/激活态的常用方法
+  LevelListDrawable，通过xml定义`<level-list>`，里面有多个`<item>`来实现类似于wifi信号强度这样的drawable，例子：
    ```xml
    <level-list xmlns:android="http://schemas.android.com/apk/res/android">
      <item android:maxLevel="0" android:drawable="@drawable/ic_wifi_signal_1" />
      <item android:maxLevel="1" android:drawable="@drawable/ic_wifi_signal_2" />
      <item android:maxLevel="2" android:drawable="@drawable/ic_wifi_signal_3" />
      <item android:maxLevel="3" android:drawable="@drawable/ic_wifi_signal_4" />
    </level-list>
    ```
    ImageView有`setImageLevel(int)`方法，可以设置显示哪个强度
+  TransitionDrawable，通过xml定义`<transition>`，里面有多个`<item>`，类似于`<layer-list>`，但是支持不同layer之间的淡入淡出，通过TransitionDrawable的`startTransition(int)`、`resetTransition()`来显示某一层
+  InsetDrawable，把它设置为一个View的背景时，可以与View的背景区域小于其bound，但是应用场景呢？
+  ClipDrawable，把源drawable裁剪后显示，可以通过随时间改变裁剪区域来做出图片逐渐展开的效果
+  ScaleDrawable，把源drawable缩放后显示，类似还有RotateDrawable，GradientDrawable
+  ShapeDrawable，定义基本几何类型
  
## [AndroidFillableLoaders](http://jorgecastillo.xyz/2015/08/16/android-fillable-loaders/)
+  PNG可以导出为SVG
+  通过SVGParser，可以将SVG指令转化为Path的指令，并将其绘制到Path对象中
+  DashPathEffect可以达到绘制加速边缘的效果
+  填满的动画思路类似于[WashingMachineView](https://github.com/naman14/WashingMachineView/)