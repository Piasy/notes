# Best Practices for Interaction and Engagement

## Designing Effective Navigation
+  Planning Screens and Their Relationships
  +  实体关系图（Entity-relationship diagram），数据、用户之间的关系、操作
  +  详尽的用例场景
  +  界面关系图
+  Planning for Multiple Touchscreen Sizes
  +  界面组合技术，在平板电脑、电视上，屏幕很大，可以把列表界面和详情界面左右排布一起显示
+  Providing Descendant and Lateral Navigation
  +  Descendant，父界面前往子界面
  +  Lateral，兄弟界面之间的跳转
  +  list和section
+  Providing Ancestral and Temporal Navigation
  +  Temporal（时间性），按下返回键，应该回到上一界面
  +  Ancestral（父子性），ActionBar/ToolBar上的返回按钮，返回父界面（通常是上一界面，但并不全是，容纳WebView的Activity就是很好的例子），需要注意的是，一定要清除backstack
+  ...

## Implementing Effective Navigation
+  Creating Swipe Views with Tabs
  +  `ViewPager`，`PagerTitleStrip`，`FragmentPagerAdapter`，`FragmentStatePagerAdapter`
+  Creating a Navigation Drawer
  +  drawer layout里面可以显式一个菜单列表
  +  `ActionBarDrawerToggle`（appcompat-v7中）可以监听drawer的开启与关闭事件，也可以用代码控制drawer的开启与关闭
  +  [完整样例](http://developer.android.com/intl/zh-cn/training/implementing-navigation/nav-drawer.html)
+  Providing Up Navigation
  +  首先在manifest里面为activity声明其parent activity，用于up navigation
  +  然后在`onOptionsItemSelected`回调中处理up navigation：
  
  ```java
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
        // Respond to the action bar's Up/Home button
        case android.R.id.home:
            Intent upIntent = NavUtils.getParentActivityIntent(this);
            if (NavUtils.shouldUpRecreateTask(this, upIntent)) {
                // This activity is NOT part of this app's task, so create a new task
                // when navigating up, with a synthesized back stack.
                TaskStackBuilder.create(this)
                        // Add all of this activity's parents to the back stack
                        .addNextIntentWithParentStack(upIntent)
                        // Navigate up to the closest parent
                        .startActivities();
            } else {
                // This activity is part of this app's task, so simply
                // navigate up to the logical parent activity.
                NavUtils.navigateUpTo(this, upIntent);
            }
            return true;
        }
        return super.onOptionsItemSelected(item);
    }
  ```

+  Providing Proper Back Navigation
  +  安卓系统都有一个物理返回键，所以不应该在UI上额外添加一个返回键
  +  安卓系统的back stack通常情况下都可以应对back navigation
  +  但是以下情况需要特殊考虑
    +  从通知栏消息、widget、navigation drawer直接进入一个深层次的activity
    
    ```java
        // Intent for the activity to open when user selects the notification
        Intent detailsIntent = new Intent(this, DetailsActivity.class);

        // Use TaskStackBuilder to build the back stack and get the PendingIntent
        PendingIntent pendingIntent =
                TaskStackBuilder.create(this)
                                // add all of DetailsActivity's parents to the stack,
                                // followed by DetailsActivity itself
                                .addNextIntentWithParentStack(upIntent)
                                .getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);

        NotificationCompat.Builder builder = new NotificationCompat.Builder(this);
        builder.setContentIntent(pendingIntent);
    ```
    
    +  fragment之间的导航
    
    ```java
        // Works with either the framework FragmentManager or the
        // support package FragmentManager (getSupportFragmentManager).
        getSupportFragmentManager().beginTransaction()
                                .add(detailFragment, "detail")
                                // Add this transaction to the back stack
                                .addToBackStack()
                                .commit();
    ```
    
    需要注意的是，当fragment是在ViewPager中水平切换时，不应该把transaction 加入到 backstack中。
    
    +  WebView
    
    ```java
        @Override
        public void onBackPressed() {
            if (mWebView.canGoBack()) {
                mWebView.goBack();
                return;
            }

            // Otherwise defer to system default behavior.
            super.onBackPressed();
        }
    ```

+  Implementing Descendant Navigation
  +  通常都是`Intent`加上`startActivity(intent)`，或者`FragmentTransaction`来完成向子界面的导航
  +  需要注意的是，如果app将打开其他app的界面，为了防止用户中途离开，再次从launcher打开app时，却是其他app的界面，可以为intent设置`FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET` flag
  
  ```java
    Intent externalActivityIntent = new Intent(Intent.ACTION_PICK);
    externalActivityIntent.setType("image/*");
    externalActivityIntent.addFlags(
            Intent.FLAG_ACTIVITY_CLEAR_WHEN_TASK_RESET);
    startActivity(externalActivityIntent);
  ```

+  Notifying the User
  +  Building a Notification
    +  使用[`NotificationCompat.Builder`](http://developer.android.com/reference/android/support/v4/app/NotificationCompat.Builder.html)创建通知栏消息
    +  响应用户对通知栏消息的点击，启动相应Activity，同时要考虑是否提供返回的导航设计
  +  Preserving Navigation when Starting an Activity
    +  点击通知栏消息，可能会启动两种类型的Activity：正常使用流程会启动的Activity；仅仅是把通知栏消息展开的Activity；
    +  前者通常需要提供back导航支持，在manifest中声明Activity父子关系，同时构造PendingIntent时构造back stack，使用`stackBuilder.getPendingIntent`创建PendingIntent
    +  后者通常不需要加入到back stack中，无需手动构造back stack，使用`PendingIntent.getActivity`创建PendingIntent，同时在manifest中设置以下选项：
        ```xml
        <activity
            android:name=".ResultActivity"
        ...
            android:launchMode="singleTask"
            android:taskAffinity=""
            android:excludeFromRecents="true">
        </activity>
        ```
  +  完整示例：
    
    ```java 
    Intent resultIntent = new Intent(this, ResultActivity.class);
    TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
    // Adds the back stack
    stackBuilder.addParentStack(ResultActivity.class);
    // Adds the Intent to the top of the stack
    stackBuilder.addNextIntent(resultIntent);
    // Gets a PendingIntent containing the entire back stack
    PendingIntent resultPendingIntent =
            stackBuilder.getPendingIntent(0, PendingIntent.FLAG_UPDATE_CURRENT);
            
    NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(this)
        .setSmallIcon(R.drawable.notification_icon)
        .setContentTitle("My notification")
        .setContentText("Hello World!")
        .setContentIntent(resultPendingIntent);

    // Sets an ID for the notification
    int mNotificationId = 001;
    // Gets an instance of the NotificationManager service
    NotificationManager mNotifyMgr = 
            (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
    // Builds the notification and issues it.
    mNotifyMgr.notify(mNotificationId, mBuilder.build());
    ```
    
  +  Updating Notifications
    +  创建通知栏消息时，如果指定了id，以后可以通过id对已经显示的消息进行修改/更新、移除操作；
    +  修改/更新只需以相同的id，新的`Notification`对象，调用`mNotifyMgr.notify`即可；
    +  如果消息支持被移除，则用户可以手动移除（单个或者所有），`setAutoCancel()`会让消息在用户点击时自动消失；`cancel(id)`和`cancelAll()`可以代码移除通知栏消息；
  +  Using Big View Styles
    +  Android 4.1之后，通知栏消息引入了action的支持，方便用户快捷操作
    
    ```java
    // Sets up the Snooze and Dismiss action buttons that will appear in the
    // big view of the notification.
    Intent dismissIntent = new Intent(this, PingService.class);
    dismissIntent.setAction(CommonConstants.ACTION_DISMISS);
    PendingIntent piDismiss = PendingIntent.getService(this, 0, dismissIntent, 0);

    Intent snoozeIntent = new Intent(this, PingService.class);
    snoozeIntent.setAction(CommonConstants.ACTION_SNOOZE);
    PendingIntent piSnooze = PendingIntent.getService(this, 0, snoozeIntent, 0);

    // Constructs the Builder object.
    NotificationCompat.Builder builder =
            new NotificationCompat.Builder(this)
            .setSmallIcon(R.drawable.ic_stat_notification)
            .setContentTitle(getString(R.string.notification))
            .setContentText(getString(R.string.ping))
            .setDefaults(Notification.DEFAULT_ALL) // requires VIBRATE permission
            /*
            * Sets the big view "big text" style and supplies the
            * text (the user's reminder message) that will be displayed
            * in the detail area of the expanded notification.
            * These calls are ignored by the support library for
            * pre-4.1 devices.
            */
            .setStyle(new NotificationCompat.BigTextStyle()
                    .bigText(msg))
            .addAction (R.drawable.ic_stat_dismiss,
                    getString(R.string.dismiss), piDismiss)
            .addAction (R.drawable.ic_stat_snooze,
                    getString(R.string.snooze), piSnooze);
    ```

  +  Displaying Progress in a Notification
    +  `setProgress (int max, int progress, boolean indeterminate)`可以为通知栏消息设置进度条，支持实时进度、持续模式