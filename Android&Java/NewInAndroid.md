#各个安卓版本引入的主要新特性

##Android 6.0 Marshamallo
+  [Runtime Permission System](http://android-developers.blogspot.jp/2015/08/building-better-apps-with-runtime.html)
  +  Step 1: check the platform, `Build.VERSION.SDK_INT >= Build.VERSION_CODES.M`
  +  Step 2: check the permission status, [`checkSelfPermission()`](http://goo.gl/T7vE7b)
  +  Step 3: explain the permission, [`shouldShowRequestPermissionRationale()`](http://goo.gl/bFyfVj)
  +  Step 4: request the permission, [`requestPermissions()`](http://goo.gl/yNuizg)
  +  Step 5: handle the response, `onRequestPermissionResult()`
+  [App Linking](https://developer.android.com/preview/features/app-linking.html)：通过注册，系统对链接的处理将直接打开官方（注册）APP，而不是显示对话框，[解读blog](https://chris.orr.me.uk/android-app-linking-how-it-works/)；
+  [Auto Backup for Apps](http://developer.android.com/preview/backup/index.html)
+  [基于百分比的Layout](https://developer.android.com/reference/android/support/percent/package-summary.html)

##5.0 Lollipop
+  Material design：包括视觉、移动、交互、主题、widgets、阴影、动效等；
+  Notifications：包括锁屏界面、优先级通知、云同步通知；

##4.4 KitKat