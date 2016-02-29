# Best Practices for Background Jobs

## Creating a Background Service
+  `IntentService`被设计于用来在一个单独的后台线程执行耗时操作，它是对Service组件的一个封装，
通过`HandlerThread`和`ServiceHandler`将其任务代码放到单独的线程执行
+  处理的结果需要通过EventBus或者LocalBroadcast发送到Activity
+  由于Service同时只会存在一个实例，所以如果同时发送多个Intent，它们会串行执行

## Loading Data in the Background
+  CursorLoader可用于从ContentProvider异步加载数据

## Managing Device Awake State
+  保持屏幕不锁屏：在Activity的onCreate函数中执行：`getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);`
+  或者在layout中加入：`android:keepScreenOn="true"`
+  保持CPU运行：wake lock
  +  例如：后台Service在锁屏后仍希望进行数据处理
  +  声明权限：`<uses-permission android:name="android.permission.WAKE_LOCK" />`
  +  获得wake lock
    
    ```java
        PowerManager powerManager = (PowerManager) getSystemService(POWER_SERVICE);
        WakeLock wakeLock = powerManager.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK,
                "MyWakelockTag");
        wakeLock.acquire();
        // do sth
        wakelock.release();
    ```
    
    +  IntentService结合WakefulBroadcastReceiver，可以有效提高后台任务的电池效率
+  Scheduling Repeating Alarms，使用`AlarmManager`
