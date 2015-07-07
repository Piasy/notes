#一些很棒的点子

##系统API
+  自Android 5.0之后，用户的“最近任务”（recent tasks）视图，可以自定义了，支持自定义图标、标题、顶栏色彩；参考：[developers](https://developer.android.com/guide/components/recents.html)，[blog](https://www.bignerdranch.com/blog/polishing-your-Android-overview-screen-entry/?utm_source=Android+Weekly&utm_campaign=22c3800806-Android_Weekly_130&utm_medium=email&utm_term=0_4eb677ad19-22c3800806-337892465)；
+  全新的Android编译系统：[Jack & Jill](http://tools.android.com/tech-docs/jackandjill)
+  [使用LinearLayout的divider属性，设置为shape drawable，控制其子元素之间的间距](http://cyrilmottier.com/2014/11/17/grid-spacing-on-android/?utm_source=Android+Weekly&utm_campaign=22c3800806-Android_Weekly_130&utm_medium=email&utm_term=0_4eb677ad19-22c3800806-337892465)
+  使用wedget，在桌面上显示内容。[示例](http://ptrprograms.blogspot.com/2014/11/building-widget-to-silence-phone.html?utm_source=Android+Weekly&utm_campaign=22c3800806-Android_Weekly_130&utm_medium=email&utm_term=0_4eb677ad19-22c3800806-337892465)
+  [Android 5.0引入TextView的CSS样式fontFeatureSettings](http://blog.sqisland.com/2014/11/android-stacked-fractions.html?utm_source=Android+Weekly&utm_campaign=22c3800806-Android_Weekly_130&utm_medium=email&utm_term=0_4eb677ad19-22c3800806-337892465)
+  [利用Action Intent尽可能利用用户手机上已有的APP功能，还不需要相关的权限](http://ryanharter.com/blog/2014/11/26/whats-your-intent/?utm_source=Android+Weekly&utm_campaign=22c3800806-Android_Weekly_130&utm_medium=email&utm_term=0_4eb677ad19-22c3800806-337892465)
+  [TextView的高级玩法：CompoundDrawable，shadow，Typeface自定义字体，Shader，HTML渲染（支持自定义tag）、Span（SpannableString：字符级别、段落级别、对其），自定义Span（立式分数、彩虹效果、彩虹动效、可点击URL、Emoji...）](http://chiuki.github.io/advanced-android-textview/)，[用xml定义drawable动画](http://chiuki.github.io/advanced-android-textview/#/3)
	![AdvancedTextView.png](assets/AdvancedTextView.png)
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

##Material design
+  [Material design中的Snackbar](https://github.com/nispok/snackbar/)，[带有Context的Toast：Crouton](https://github.com/keyboardsurfer/Crouton)


##有意思的第三方库
+  [基于UDP组播的Intent发送和接收](http://www.androidzeitgeist.com/2014/11/introducing-android-network-intents17.html?utm_source=Android+Weekly&utm_campaign=a94f126150-Android_Weekly_129&utm_medium=email&utm_term=0_4eb677ad19-a94f126150-337892465)
