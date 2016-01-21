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
