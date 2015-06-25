#Google dev 100 days系列视频

##Day 1：[The Android Design Library](http://android-developers.blogspot.com/2015/05/android-design-support-library.html)
compile 'com.android.support:design:22.2.0'
+  TextInputLayout  
EditText的hint将一直以浮动形式显示在输入框下面，并且可以通过setError()接口，提示错误信息；
+  FloatingActionButton  
浮动操作按钮，默认设置为app theme的颜色；可以设置为mini大小；继承自ImageView，可以为其设置任何显示内容；
+  Snackbar  
轻量、快速的用户反馈方式；显示在屏幕底部，支持一个可选的操作按钮，显示超时后自动消失，用户也可以滑动使其消失；API与toast基本一致，但功能更强大；
```java
Snackbar
  .make(parentLayout, R.string.snackbar_text, Snackbar.LENGTH_LONG)
  .setAction(R.string.snackbar_action, myOnClickListener)
  .show(); // Don’t forget to show!
```
+  TabLayout  
多view tab支持；可以和ViewPager一起使用；
+  NavigationView  
支持通过menu资源文件创建导航视图；把NavigationView放到DrawerLayout里面，通过`app:headerLayout`属性设置headerLayout，通过`app:menu`属性设置导航菜单内容，支持高亮显示当前选中的菜单项，支持多级菜单，通过`setNavigationItemSelectedListener()`接口设置菜单点击回调；需要注意的是NavigationView会负责状态栏的操作，在API 21+时，需要考虑状态栏的控制；
+  CoordinatorLayout
  +  floating action buttons  
  将FloatingActionButton放到CoordinatorLayout中，在使用Snackbar时，将CoordinatorLayout传入到Snackbar的`Snackbar.make()`函数中，那么action button将会在snack bar显示和消失时，自动改变其位置，不需要任何代码；`layout_anchor`和`layout_anchorGravity`属性可以设置浮动view和其他view的位置；
  +  app bar(以前的ActionBar)  
  AppBarLayout可以和RecyclerView响应，RecyclerView通过`app:layout_behavior`属性指定Behavior子类，AppBarLayout的元素通过`app:layout_scrollFlags`属性指定对滑动事件的响应方式；
  +  Collapsing Toolbars  
  把Toolbar直接作为AppBarLayout的子元素，无法满足Toolbar的不同元素以不同方式响应滑动操作的需求，为此，可以在中间加一层CollapsingToolbarLayout；可以实现顶部自定义布局，随滚动而变化为bar，再滚动则消失，反向滚动则显示的炫酷效果  
  !video[patterns-scrolling-techniques_flex_space_image_xhdpi_003.webm](assets/patterns-scrolling-techniques_flex_space_image_xhdpi_003.webm)  
  +  Custom views  
  自定义View可以通过定义[Coordinator.Behavior](http://developer.android.com/reference/android/support/design/widget/CoordinatorLayout.Behavior.html)的子类来与CoordinatorLayout进行合作，从而实现自定义的显示效果；
  
##Day 2：[LRUCache](http://developer.android.com/reference/android/util/LruCache.html)
Android framework实现的LRU缓存算法类，对于Bitmap的使用场景非常合适；

##Day 3：[Google Play Services 7.5](http://android-developers.blogspot.com/2015/05/a-closer-look-at-google-play-services-75.html)
+  [Smart Lock for Passwords](https://developers.google.com/identity/smartlock-passwords/android/)  
密码保存服务API；
+  Instance ID, Identity, and Authorization  
提供应用识别、授权服务；
+  [Google Cloud Messaging](https://developers.google.com/cloud-messaging/)  
消息推送、上报服务、基于话题订阅的推送、网络请求管理（优化）；
+  [App invite](https://developers.google.com/app-invites)  
集成的邀请新用户（分享APP）功能；
+  [Google Cast](http://www.google.com/cast/)  
谷歌提供的多设备视频、音频播放功能；
+  Android Wear、Google Fit...

##Day 4：[Web app的推送通知](https://www.youtube.com/watch?v=Z_K8QPQe6oM)

##Day 5：[用户活动识别：步行、跑步、骑车等](https://developers.google.com/android/reference/com/google/android/gms/location/ActivityRecognitionApi)

##Day 6：[Android wear使用MAP API](https://developers.google.com/maps/documentation/android/wear)

##Day 7：[语音互动，不仅是启动命令，还能交互](http://io2015codelabs.appspot.com/codelabs/voice-interaction#1)

##Day 8：[谷歌地图的Lite模式，高性能显示多张静态地图](https://developers.google.com/maps/documentation/android/lite)

##Day 9：[使用Lint工具进行代码检查：性能、bug、内存泄漏等](http://developer.android.com/tools/debugging/improving-w-lint.html)

##Day 10：[逐步优化网页响应的速度](https://youtu.be/d5_6yHixpsQ)

##Day 11：[指纹验证API](https://youtu.be/VOn7VrTRlA4)

##Day 12：[Google Play商店的icon、描述文字-安装量统计功能](https://youtu.be/B6ydLpkhq04)

##Day 13：[优化APP在google搜索中的结果](https://youtu.be/NrcCUaAaHG4)

##Day 14：[Google play for work：为办公设备订制的APP](https://youtu.be/dH41OutAMNM)

##Day 15：[Google map的AnimateCamera API](https://youtu.be/gkze68UlHdk)

##Day 16：[Transparency & Alpha使用时的渲染性能优化](https://youtu.be/wIy8g8yNhNk)
+  在动画开始时，`setLayerType(View.LAYER_TYPE_HARDWARE, null)`，动画结束后，`setLayerType(View.LAYER_TYPE_NONE, null)`；在API >= 16时，可以只调用`ViewPropertyAnimator.alpha(0.0f).withLayer()`接口即可；
+  使用shadow时，重写View的`hasOverlappingRendering()`接口，返回false；
+  只有当确定瓶颈是这部分view的渲染时，才有必要这样优化；

##Day 17：[Google Analytics SDK](https://youtu.be/ecAzIIvbKIU)
下载量、安装量、用户离开APP的页面、地区分布等多种统计信息；

##Day 18：[AdMob：APP广告分发平台](https://youtu.be/N-EXqPBaaKQ)
多种展示方式、自动获取费用最高的广告、Google Analytics集成等特点；

##Day 19：[designing for drivers](https://youtu.be/vG9c5egwEYY)
6步法则；只展示驾驶员真正想要的内容；使用语音控制；

##Day 20：[Android M中的Android for work API](https://youtu.be/vcSj8ln-BlE)