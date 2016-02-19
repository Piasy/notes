# Best Practices for User Input

## Using Touch Gestures
+  手势检测
  +  监听touch事件
  +  根据当前及历史touch事件，判断是否符合支持的手势
  +  使用`MotionEventCompat`的辅助方法从`MotionEvent`类中读取信息，
  使用`GestureDetectorCompat`进行兼容性的手势检测
  +  `MotionEvent`类包含了触点id（多点触控支持），位置，压力等众多信息，
  用于进行手势检测
  +  使用`GestureDetector`/`GestureDetectorCompat`类进行手势检测，
  传入`GestureDetector.OnGestureListener`实现类，在View/Activity的
  `onTouchEvent`中把`MotionEvent`对象传入detector，系统将会在特定手势
  触发时调用回调，例如：fling, long press等
  +  [示例代码](http://developer.android.com/training/gestures/detector.html#detect)
+  移动追踪
  +  会有一个touch slop的概念，触点在屏幕上移动超过一定距离才会被认为是移动
  +  常见的手势检测方式有：起止点，移动方向，历史轨迹，移动速度等
  +  速度追踪可以使用`VelocityTracker`和`VelocityTrackerCompat`类，
  [代码示例](http://developer.android.com/training/gestures/movement.html#velocity)
+  为滚动手势添加动效
  +  大多数情况下，内容所需区域大于view大小的，都应该使用ScrollView或者
  ListView/RecyclerView，把滚动的处理交给系统
  +  但也有特殊情况，例如实现滚动手势时，如果有同样的效果，体验会更好，
  此时就需要使用`Scroller`或者`OverScroller`了
  +  更建议使用`OverScroller`，因为它的兼容性更好，此外，对于内容的滚动需求，
  应该使用`ScrollView`或者`HorizontalScrollView`
  +  scroller和安卓平台独立，只用于记录位置随着滚动的变化，使用者需要按照一定
  的频率去获取位置并应用到view上，以得到平滑的滚动效果
  +  滚动的不同类型
    +  dragging，滚动内容的过程中手指不离开屏幕，只需重写
    `GestureDetector.OnGestureListener::onScroll()`方法即可
    +  flinging，滚动内容的时候，快速滚动后手指离开屏幕，内容仍需保持一定速度继续滚动，
    但是逐渐减速直至停止，这种情况下需要重写`GestureDetector.OnGestureListener::
    onFling()`且结合scroller
  +  [fling结合scroller的示例](http://developer.android.com/training/
  gestures/scroll.html##scroll)
+  多点触控
  +  onTouchEvent回调的MotionEvent对象中，包含了多触点的信息
  +  额外触点的按下、抬起事件：`ACTION_POINTER_DOWN`, `ACTION_POINTER_UP`
+  拖拽和缩放
  +  3.0以上版本有`View.OnDragListener`可供使用，包括drag drop 阴影等
  +  使用`ScaleGestureDetector`来进行view的缩放支持
+  ViewGroup的点击事件
  +  使用`onInterceptTouchEvent()`函数拦截对子View的点击事件
  +  使用`ViewConfiguration`类获取系统定义的一系列常量：间距，速度，时间等
  +  使用`TouchDelegate`类，扩展子View的点击热区，使其点击热区大于子View的边界

## Handling Keyboard Input
+  EditText等输入控件可以通过`inputType`属性设置输入类型，一方面控制输入法的输入模式，
另一方面限制输入内容
+  拼写检查、自动纠正以及Input Method Action
  +  `android:inputType="textCapSentences|textAutoCorrect"`
  +  `android:imeOptions="actionSend"`, `actionSearch`
  +  `editText.setOnEditorActionListener`
+  键盘可见性变化
  +  manifest中可以配置Activity启动时就显示键盘：`android:windowSoftInputMode="stateVisible"`
  +  代码主动显示键盘
  
   	```java
    public void showSoftKeyboard(View view) {
        if (view.requestFocus()) {
            InputMethodManager imm = (InputMethodManager)
                    getSystemService(Context.INPUT_METHOD_SERVICE);
            imm.showSoftInput(view, InputMethodManager.SHOW_IMPLICIT);
        }
    }	
    ```

  +  此外，manifest中通过`windowSoftInputMode`来控制键盘弹起时Layout的变化：
  `adjustResize`, `adjustPan`, `adjustUnspecified`, `adjustNothing`
  +  此外，在activity的mode不为`adjustNothing`时，[KeyboardVisibilityEvent](https://
  github.com/yshrsmz/KeyboardVisibilityEvent)库可以检测键盘弹出和收起事件，也提供了
  手动弹出和收起的方法
+  键盘导航：主要用于外接键盘的操作，并非跳转，而是焦点移动
  +  tab键：View的`android:nextFocusForward`属性，指定当前View在获得焦点状态下，按下tab键，
  焦点移动的下一个View的id
  +  方向键：`android:nextFocusUp`, `android:nextFocusDown`, `android:nextFocusLeft`, 
  `android:nextFocusRight`
+  直接响应键盘按键：Activity等类实现了`KeyEvent.Callback`接口可以监听键盘事件
