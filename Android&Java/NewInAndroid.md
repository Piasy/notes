#各个安卓版本引入的主要新特性

##Android 6.0 Marshamallo
+  [Runtime Permission System](http://android-developers.blogspot.jp/2015/08/building-better-apps-with-runtime.html)
  +  Step 1: check the platform, `Build.VERSION.SDK_INT >= Build.VERSION_CODES.M`
  +  Step 2: check the permission status, [`checkSelfPermission()`](http://goo.gl/T7vE7b)，只可能返回两种状态：已授权，未授权；不存在“未请求”状态；
  +  Step 3: explain the permission, [`shouldShowRequestPermissionRationale()`](http://goo.gl/bFyfVj)，官方文档说“显示Rationale是为了解释不那么明显的权限请求，该方法就是检查是否需要显示Rationale”，那么到底是检查是不是“不那么明显”呢？还是检查是否已授权呢？显然应该是后者，系统怎么可能检查什么“不那么明显”？那么问题是这个方法和`checkSelfPermission`有什么区别呢？因为后者是异步的？还是说因为这是两种不同的使用场景？
  +  Step 4: request the permission, [`requestPermissions()`](http://goo.gl/yNuizg)，不能保证一定被授权；而且此过程Activity可能会被pause，有些权限的授予甚至会要求重启APP进程；如果设置了manifest的Activity标签中设置了`android:noHistory="true"`，将不能请求权限，因为此Activity收不到任何回调；
  +  Step 5: handle the response, `onRequestPermissionResult()`
  +  TODO：测试，shouldShowRequestPermissionRationale和checkSelfPermission在效果上有没有什么区别？未请求过权限和授权被拒绝，这两个函数的效果有何区别？每次都选择拒绝授权，每次都调用requestPermissions，会是什么效果？是否第二次拒绝后就不会弹对话框了？那么如何判断是否请求过权限？SharedPreference？测试完成后完成博客一篇。
+  [App Linking](https://developer.android.com/preview/features/app-linking.html)：通过注册，系统对链接的处理将直接打开官方（注册）APP，而不是显示对话框，[解读blog](https://chris.orr.me.uk/android-app-linking-how-it-works/)；
+  [Auto Backup for Apps](http://developer.android.com/preview/backup/index.html)
+  [基于百分比的Layout](https://developer.android.com/reference/android/support/percent/package-summary.html)

##5.0 Lollipop
+  Material design：包括视觉、移动、交互、主题、widgets、阴影、动效等；
+  Notifications：包括锁屏界面、优先级通知、云同步通知；

##4.4 KitKat