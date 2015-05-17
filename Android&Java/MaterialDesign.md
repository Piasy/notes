##Material Design

#[Ripples](https://blog.stylingandroid.com/ripples-part-1/)
```xml
<?xml version="1.0" encoding="utf-8"?>
<ripple 
  xmlns:android="http://schemas.android.com/apk/res/android"
  android:color="?android:colorControlHighlight" />
```
+  实现点击态、正常态的视觉切换（反馈）
  +  使用多个drwable，通过selector实现；
  +  通过新的ripple api实现点击反馈；
+  动画可能超出parent layout的范围；
+  可以通过为ripple加一个item来限制动画的范围；
+  在ripple中使用theme的元素定义color，可以保证整个app中view颜色的一致性；
+  `android:colorPrimary`（AppCompat中使用`colorPrimary`）定义ActionBar的颜色；
+  `android:colorPrimaryDark`（AppCompat中使用`colorPrimaryDark`）定义状态栏的颜色（5.0之前无效）；
+  直接Activity会导致ActionBar不显示，应继承自AppCompatActivity（或ActionBarActivity，已弃用）；

##AppCompat
+  在5.0之前使用Material Design的兼容库；
+  AppCompat不包括ripple，因为ripple使用了5.0引入的render thread去进行渲染；

##TextAppearance
+  `android.R.style.TextAppearance.Material.*`（AppCompat使用`R.style.TextAppearance.AppCompat.*`）;

##CardView
+  支持圆角、阴影的卡片式View，继承自FrameLayout
+  在5.0以前，通过为CardView设置额外的padding来绘制阴影；因此，如果想要去掉阴影区域额外的padding，目前只能通过将`contentPaddingXXXX`属性设为负值来实现；

##RecyclerView
+  ListView默认有selector overlay，点击有视觉效果；RecyclerView需要使用`RecyclerView.ItemDecoration`手动实现（ItemDecoration还可以做更多事情，例如divider等）；

##RecyclerView的item支持拖拽

##Activity切换动画
+  `ActivityOptionsCompat.makeSceneTransitionAnimation`，`ActivityCompat.finishAfterTransition`