#一些很棒的点子

##系统API
+  自Android 5.0之后，用户的“最近任务”（recent tasks）视图，可以自定义了，支持自定义图标、标题、顶栏色彩；参考：[developers](https://developer.android.com/guide/components/recents.html)，[blog](https://www.bignerdranch.com/blog/polishing-your-Android-overview-screen-entry)；
+  全新的Android编译系统：[Jack & Jill](http://tools.android.com/tech-docs/jackandjill)
+  [使用LinearLayout的divider属性，设置为shape drawable，控制其子元素之间的间距](http://cyrilmottier.com/2014/11/17/grid-spacing-on-android)
+  使用wedget，在桌面上显示内容。[示例](http://ptrprograms.blogspot.com/2014/11/building-widget-to-silence-phone.html)
+  [Android 5.0引入TextView的CSS样式fontFeatureSettings](http://blog.sqisland.com/2014/11/android-stacked-fractions.html)
+  [利用Action Intent尽可能利用用户手机上已有的APP功能，还不需要相关的权限](http://ryanharter.com/blog/2014/11/26/whats-your-intent)
+  [TextView的高级玩法：CompoundDrawable，shadow，Typeface自定义字体，Shader，HTML渲染（支持自定义tag）、Span（SpannableString：字符级别、段落级别、对其），自定义Span（立式分数、彩虹效果、彩虹动效、可点击URL、Emoji...）](http://chiuki.github.io/advanced-android-textview/)，[用xml定义drawable动画](http://chiuki.github.io/advanced-android-textview/#/3)
	![AdvancedTextView.png](assets/AdvancedTextView.png)
+  [Android integration of multiple icon providers such as FontAwesome, Entypo, Typicons,...](https://github.com/JoanZapata/android-iconify)
+  [Shape Drawable：形状、圆角、边框、填充、渐变色填充等](http://trinea.iteye.com/blog/1483949)
+  [View绘制时加上特效：Shader，图像渲染、线性渐变、环形渐变、扫描渐变、组合渐变](http://blog.csdn.net/ldj299/article/details/6166071)
+  [安卓系统的“售货亭模式”](http://cases.azoft.com/android-kiosk-mode-rules-restrictions/)
  +  只允许用户在一个应用程序内使用，不能接收到系统通知，状态栏，退出程序，类似于ATM机，只能使用一个应用程序。
  +  5.0：设置菜单内Screen pinning mode；`startLockTask()`；
  +  pre 5.0：
    +  自启动：监听启动事件`android.intent.action.BOOT_COMPLETED`，随系统启动APP；
	+  监听返回键，并且不返回；
	+  manifest文件启动activity添加三个category：`android.intent.category.HOME`、`android.intent.category.LAUNCHER`、`android.intent.category.DEFAULT`；
	+  电源键：只在4.0以下的系统可以达到效果  
	```java
	@Override  
	public void onAttachedToWindow() {  
	    getWindow().addFlags(  
	        WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD);  
	    getWindow().addFlags(  
	        WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED);  
	}
	```
	+  系统对话框：监听失去焦点事件，然后发广播关掉所有系统对话框  
	```java
	@Override  
	public void onWindowFocusChanged(boolean hasFocus) {  
	  super.onWindowFocusChanged(hasFocus);  
	  if(!hasFocus) {  
	    Intent closeDialog =   
	          new Intent(Intent.ACTION_CLOSE_SYSTEM_DIALOGS);  
	    sendBroadcast(closeDialog);  
	  }  
	}  
	```
	+  虚拟键盘
	+  状态栏：全屏、TYPE_SYSTEM_ALERT、截取状态栏区域的点击事件
+  onResumeFragments
  +  FragmentActivity的子类（AppCompatActivity等）均有此lifecycle方法
  +  FragmentActivity的onResume函数调用的时候，所有的Fragment都还没有onResume，Activity并不能保证保存的状态已被恢复，而在这种情况下，是不能进行fragment的transaction的，而在onResumeFragments则能保证调用的时候activity已经恢复了状态
  +  [read more](Fragments.md#)
+  MultiDex  
MultiDex会导致build变慢，在Dalvik虚拟机上（未使用ART技术时），APP启动速度也会变慢，因为ClassLoader要从第二个（甚至更多个）dex文件中加载类；  
有时候还会导致build过程中的dex步骤报OOM错误；新提出的Jack&Jill构建技术将解决这一问题；  
```java
UNEXPECTED TOP-LEVEL ERROR:
java.lang.OutOfMemoryError: GC overhead limit exceeded
  at com.android.dx.cf.code.ExecutionStack.copy(ExecutionStack.java:66)
  at 
  ...
```  
在build.gradle中加入以下片段即可解决：  
```groovy
android {
  // ...
  dexOptions {
    javaMaxHeapSize “2048M”
  }
}
```
+  [缩小APK包体积的Tips](http://cyrilmottier.com/2014/08/26/putting-your-apks-on-diet/)
  +  Proguard
  +  Lint
  +  不必为每种dpi打包资源文件（图标）
  +  移除第三方库中不必要的资源文件  
  ```groovy
	defaultConfig {
	    resConfigs "en", "de", "fr", "it"
	    resConfigs "nodpi", "hdpi", "xhdpi", "xxhdpi", "xxxhdpi"
	}
  ```
  +  图片压缩，9-patches
  +  Limit the number of architectures, armabi, x86 is enough
  +  Reuse whenever possible：图标如果只是颜色不同、旋转，则可以只打包一个，然后通过tint/tintMode/ColorFilter/RotateDrawable来重复利用
  +  Render in code when appropriate
  +  Going even further? Server side packaging，根据设备具体细节打包资源，但是有一定风险。
+  [使用ViewPager的同时不用Fragment作为显示的内容](https://www.bignerdranch.com/blog/viewpager-without-fragments)
+  [Notification中加入联系人信息之后，通知消息的显示将有更高的优先级](https://plus.google.com/+AndroidDevelopers/posts/7QBWvNXs2mD)
+  [xml中使用tools属性来辅助IDE](https://speakerdeck.com/rock3r/tools-of-the-trade-droidcon-nyc-2015)
  +  辅助lint：类似于@SuppressWarnings（tools:ignore），@TargetApi（tools:targetApi），指定locale（tools:locale）
  +  辅助预览layout：tool:context, tools:showIn, tools:menu, tools:actionBarNavMode, 指定frament的layout（tools:layout），tools:listheader/listitem/listfooter
  +  Support Annotations: @Nullable/NonNull, resources ids(), range, collection size, TypeDef, Thread(MainThread/UiThread/BinderThread/WorkerThread), "Architecture"(CallSuper/CheckResult/VisibleForTesting), Permission{  RequiresPermission(Manifest.permission.BLUTOOTH)  }, proguard(Keep)
  +  ViewDebug: @ViewDebug.ExportedProperty
+  [Speed up your app](https://speakerdeck.com/udinic/speed-up-your-app-droidcon-nyc-2015)
  +  Rules
    +  Always Measure
    +  Use[Experience] Slow Device
    +  Consider Trade-Offs
  +  Systrace
  +  Traceview
  +  Memory Profiling(AS + Eclipse MAT)
  +  Memory tips
    +  Bitmap's pixel format
    +  Context Awareness
    +  HashMap v.s. ArrayMap/Sparce*Array
  +  LeakCanary
  +  GPU Profiling
  +  Hierarchy Viewer
  +  Overdraw
  +  Alpha
    +  TextView: setTextColor() instead of setAlpha()
    +  ImageView: setImageAlpha() instead of setAlpha()
    +  CustomView: handle alpha yourself by overriding onSetAlpha(), overriding hasOverlappingRendering()
  +  Hardware Acceleration
    +  view.setLayerType(View.LAYER_TYPE_HARDWARE, null)
    +  view.animate()....withLayer().start()
+  [Percent layout library](https://developer.android.com/tools/support-library/features.html#percent)
  +  为什么要有Percent？
    +  LinearLayout的layout_weight属性可以实现按需+比例分配空间，但是当相对定位与比例分配都需要使用的时候，就不得不使用两层Layout来实现了，有损性能；另注：当LinearLayout只有一个子View使用layout_weight属性时，将需要按比例分配的长/宽属性置为0，可以提高性能，因为layout_weight被使用时会有两遍measure，而如果置为0，使用layout_weight的子View第一遍measure就可以省略。
    +  嵌套的layout_weight使用将会导致性能下降，指数级
  +  PercentRelativeLayout/PercentFrameLayout
  +  应该还是会需要两次measure，还是应该尽力避免嵌套Percent，此外，减少Layout的层数是一个常识
  +  例子  
  ```xml
    <android.support.percent.PercentRelativeLayout 
      xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity">
    
      <View
        android:id="@+id/first"
        android:background="@color/sa_green_dark"
        app:layout_heightPercent="50%"
        app:layout_marginLeftPercent="25%"
        app:layout_marginTopPercent="25%"
        app:layout_widthPercent="50%" />
    
      <View
        android:layout_width="0dp"
        android:layout_height="32dp"
        android:layout_alignLeft="@id/first"
        android:layout_alignStart="@id/first"
        android:layout_alignRight="@id/first"
        android:layout_alignEnd="@id/first"
        android:layout_below="@id/first"
        android:layout_marginTop="8dp"
        android:background="@color/light_grey" />
    
    </android.support.percent.PercentRelativeLayout>
  ```
  ![PercentRelativeLayout.png](assets/PercentRelativeLayout.png)
  +  pitfalls
    +  当子View需要的长/宽大于给定的percent时，可以通过指定layout_width/height为wrap_content来实现大小扩展，然而似乎不起效？
    +  Percent*Layout中不要使用padding，否则总大小将小于100%，可能会导致对其问题
+  [Layout animations on RecyclerView](http://antonioleiva.com/layout-animations-on-recyclerview/)
  +  RecyclerView的子类，重写`protected void attachLayoutAnimationParameters(View child, ViewGroup.LayoutParams params, int index, int count)`方法，控制layout动画的播放
  +  使用xml定义animation
+  [Chrome custom tabs](https://medium.com/ribot-labs/exploring-chrome-customs-tabs-on-android-ef427effe2f4)
+  执行定时任务，可能的实现方式有：[Alarm](http://developer.android.com/reference/android/app/AlarmManager.html), [JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler.html), API 21+, [JobSchedulerCompat](https://github.com/evant/JobSchedulerCompat) API 10+, [GcmNetworkManager](https://developers.google.com/android/reference/com/google/android/gms/gcm/GcmNetworkManager), [分享](https://plus.google.com/+AndroidDevelopers/posts/GdNrQciPwqo)。
+  xml需要预览的内容，统统用tools:属性，否则会有运行时开销，[参考](http://huteri.me/2015/07/11/beware-of-setting-image-resources-for-preview-purpose-in-xml/)

##Material design
+  [Material design中的Snackbar](https://github.com/nispok/snackbar/)，[带有Context的Toast：Crouton](https://github.com/keyboardsurfer/Crouton)

##构建/工具
+  [利用buildSrc工程和Codemodel自动生成代码](http://www.thedroidsonroids.com/blog/how-to-generate-java-sources-using-buildsrc-gradle-project)，buildSrc目录下的代码将作为gradle插件被编译，并自动添加到工程的依赖中
+  第三方库在manifest中声明的权限，可能app中并不会使用，可以[通过`uses-permission`标签的`tools:node="remove"`属性，使得gradle在进行manifest merge时，移除该权限](http://blog.forkingcode.com/2015/09/the-unexpected-permission.html)，例子：
```xml
<uses-permission 
    android:name="android.permission.WRITE_EXTERNAL_STORAGE" tools:node="remove"/>
<uses-permission 
    android:name="android.permission.READ_EXTERNAL_STORAGE" tools:node="remove"/>
```

##有意思的第三方库
+  [基于UDP组播的Intent发送和接收](http://www.androidzeitgeist.com/2014/11/introducing-android-network-intents17.html)
+  [将SQLite操作封装为rx API](http://beust.com/weblog/2015/06/01/easy-sqlite-on-android-with-rxjava/)，封装思想值得借鉴
+  [Prism](https://blog.stylingandroid.com/prism-fundamentals-part-1)，为各种部件（View，Window，StatusBar）设置颜色、背景，API简洁，功能强大；
+  [Fontinator](https://github.com/svendvd/fontinator)，自定义字体使用帮助库
+  [Quick return with CoordinatorLayout](https://medium.com/@bherbst/quick-return-with-recyclerview-e70c8da9b4c1?mc_cid=5e6ec8b400&mc_eid=fb5841ce0e)
+  [Design support library demo](https://github.com/chrisbanes/cheesesquare)
+  [FlatBuffer，比JSON更高效的序列化格式](http://frogermcs.github.io/flatbuffers-in-android-introdution/)
+  [android-iconify，图标化字体应用库，支持配置大小、颜色、动效！](http://blog.joanzapata.com/iconify-just-got-a-lot-better)
+  [Favor composition over inheritance，Adapter组合复用](https://github.com/sockeqwe/AdapterDelegates)
+  [Drawble上加蒙色，减小包大小](http://andraskindler.com/blog/2015/tinting_drawables/)

##Google API
+  [Nearby API](https://developers.google.com/nearby/)
+  [Google play service条形码/二维码识别](http://android-developers.blogspot.co.uk/2015/08/barcode-detection-in-google-play.html)
+  [Google play service人脸识别](http://android-developers.blogspot.co.uk/2015/08/face-detection-in-google-play-services.html)
+  [Google Eddystone with the Proximity Beacon API](https://medium.com/ribot-labs/exploring-google-eddystone-with-the-proximity-beacon-api-bc9256c97e05)，Beacon是一些蓝牙低能耗发射器，它们能够向附近的电子设备发射信息，提供基于附近位置的服务。

##最佳实践
+  使用[Headless Fragment](Fragments.md#使用fragment进行后台处理headless-fragment)把部分Activity公用的逻辑封装起来，避免将只被部分Activity公用的逻辑加到所有Activity的父类中。