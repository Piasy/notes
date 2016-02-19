# Best Practices for User Interface

## Designing for Multiple Screens
+  支持不同屏幕尺寸
  +  layout中尽量使用`wrap_content`或`match_parent`，而不是固定的数值
  +  资源文件可以定义不同的Qualifier，根据屏幕尺寸（dp值，最小宽高dp值，large等），dpi级别，屏幕朝向等等
  +  Nine-patch Bitmaps
  
## Adding the App Bar
+  使用Toolbar，`setSupportActionBar`设置tool bar，`getSupportActionBar`可以返回一个`ActionBar`对象（尽管设置的是tool bar），可以对app bar进行相关操作
+  app bar的action通过option menu进行设置，响应在`onOptionsItemSelected`回调中
+  up navigation：在manifest中声明parent activity，同时设置`getSupportActionBar().setDisplayHomeAsUpEnabled(true);`；如此设置之后就无需在代码中进行响应操作了；
+  action view和action provider：例如search view

## Showing Pop-Up Messages
+  Snackbar，和Toast类似，但是更符合material design
+  自动消失，可以加入action button
+  使用在CoordinatorLayout中，还能让其他View在其显示时自动上移，协同显示
  
## Creating Custom Views
+  设计良好的自定义view首先应该是一个设计良好的类，高效利用cpu和内存等
+  且应该遵循安卓的标准
+  提供xml属性，可以在layout中配置view
+  accessibility  &&  compatibility
+  在`onDraw`函数中进行自定义绘制
+  在`onSizeChanged`函数中响应尺寸的变化，layout的变化
+  `onLayout`, `onMeasure`等
+  响应输入事件：点击、滑动、fling等，`onTouchEvent`
+  `TouchEvent`的信息比较原始，通常可以使用系统提供的帮助类提供的高级API：`GestureDetector`
+  fling事件的处理需要结合`Scroller`，动画等方式，以达到真实世界的物理运动效果
+  优化view的效率
  +  减小循环被调用的函数的执行时间，例如onDraw；减少其中的内存分配；
  +  在初始化，或者动画开始前进行对象的创建，避免在动画过程中进行内存分配；
  +  减少invalidate()的调用，但是要注意，当view的内容发生变化需要更新时，必须要调用，否则不会生效；
  +  减少requestLayout()的调用，它可能会导致多次遍历整个view树；同时也应该使得布局扁平化，不要太深的嵌套；
  +  如果ui过于复杂，可以考虑自定义ViewGroup以对ui进行加速处理；这时ViewGroup的子view是满足特定条件的，可以减少measure, layout的必要；

## Creating Backward-Compatible UIs
+  通过抽象出接口，来定义UI的API，再根据不同的系统版本，选择不同的实现，以达到兼容的目的；
+  新版系统直接proxy到新的系统API，旧的版本采取自定义实现方式；

## [Implementing Accessibility](http://developer.android.com/training/accessibility/index.html)
...

## Managing the System UI
+  包括状态栏（顶部），导航栏（底部虚拟按键区域）
+  API 14+，让status bar可见性为gone，指定theme为`@style/Theme.AppCompat.Light.NoActionBar.FullScreen`或者`@android:style/Theme.Holo.NoActionBar.Fullscreen`

    ```xml
    <style name="Theme.AppCompat.Light.NoActionBar.FullScreen" parent="@style/Theme.AppCompat.Light">
        <item name="windowNoTitle">true</item>
        <item name="windowActionBar">false</item>
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowContentOverlay">@null</item>
    </style>
    ```
+  API 14+，让navigation bar可见性为gone

    ```java
    View decorView = getWindow().getDecorView();
    // Hide both the navigation bar and the status bar.
    // SYSTEM_UI_FLAG_FULLSCREEN is only available on Android 4.1 and higher, but as
    // a general rule, you should design your app to hide the status bar whenever you
    // hide the navigation bar.
    int uiOptions = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION
                | View.SYSTEM_UI_FLAG_FULLSCREEN;
    decorView.setSystemUiVisibility(uiOptions);
    ```

+  注意，上述方式有以下问题
  +  用户点击任何区域，都会导致navigation bar出现，且保持可见（属性被清除）；
  +  一旦属性被清除，需要重新设置，使得navigation bar重新隐藏；
  +  所以需要`decorView.setOnSystemUiVisibilityChangeListener`，监听system ui可见性变化，在可见时，通过postDelayed，再次隐藏之；
+  API 19+，以后，引入了`SYSTEM_UI_FLAG_IMMERSIVE`和`SYSTEM_UI_FLAG_IMMERSIVE_STICKY`，前者同样存在上述问题；
+  更完整的可以参考最新版AndroidStudio(2.0 preview 7)创建的`FullscreenActivity`模板；
+  响应system ui可见性的变化

    ```java
    View decorView = getWindow().getDecorView();
    decorView.setOnSystemUiVisibilityChangeListener
            (new View.OnSystemUiVisibilityChangeListener() {
        @Override
        public void onSystemUiVisibilityChange(int visibility) {
            // Note that system bars will only be "visible" if none of the
            // LOW_PROFILE, HIDE_NAVIGATION, or FULLSCREEN flags are set.
            if ((visibility & View.SYSTEM_UI_FLAG_FULLSCREEN) == 0) {
                // TODO: The system bars are visible. Make any desired
                // adjustments to your UI, such as showing the action bar or
                // other navigational controls.
            } else {
                // TODO: The system bars are NOT visible. Make any desired
                // adjustments to your UI, such as hiding the action bar or
                // other navigational controls.
            }
        }
    });
    ```

## Material Design for Developers
更多参见material design专题，包括：

+  material theme及其定制;
+  CardView, RecyclerView, item animation;
+  Custom shadows and view clipping;
+  Vector drawables, tint, palette;
+  Custom animations: ripple, reveal, activity transition, shared element transition, curved motion, Animate View State Changes, Animate Vector Drawables;

兼容性维护：

+  各种backport；
+  Define Alternative Styles；
+  Provide Alternative Layouts；
+  Support Library；
