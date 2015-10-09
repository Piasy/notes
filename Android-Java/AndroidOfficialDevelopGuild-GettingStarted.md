#[安卓官方开发指南](http://developer.android.com/training/index.html)

##Getting Started
+  Styling the Action Bar
  +  ActionBar可以有tab，也可以有分离的底部bar，用于显示action items
  +  使用Theme
	+  Theme.Holo，暗黑系主题
	+  Theme.Holo.Light，光明系主题
	+  Theme.Holo.Light.DarkActionBar，光明系主题，暗黑系ActionBar
	+  `<application android:theme="@android:style/Theme.Holo.Light" ... />`
	+  使用support library时
		+  Theme.AppCompat，暗黑系
		+  Theme.AppCompat.Light
		+  Theme.AppCompat.Light.DarkActionBar
	+  [Android-Action-Bar-Icons](https://github.com/svenkapudija/Android-Action-Bar-Icons)
  +  自定义背景
    +  standard: `android:actionBarStyle`  ==>  `android:background`
	+  support: `actionBarStyle`  ==>  `background`
	+  示例
	```xml
		<?xml version="1.0" encoding="utf-8"?>
		<resources>
			<!-- the theme applied to the application or activity -->
			<style name="CustomActionBarTheme"
				parent="@style/Theme.AppCompat.Light.DarkActionBar">
				<item name="android:actionBarStyle">@style/MyActionBar</item>
		
				<!-- Support library compatibility -->
				<item name="actionBarStyle">@style/MyActionBar</item>
			</style>
		
			<!-- ActionBar styles -->
			<style name="MyActionBar"
				parent="@style/Widget.AppCompat.Light.ActionBar.Solid.Inverse">
				<item name="android:background">@drawable/actionbar_background</item>
		
				<!-- Support library compatibility -->
				<item name="background">@drawable/actionbar_background</item>
			</style>
		</resources>
	```
  +  自定义文字颜色
    +  standard
	  +  Action bar title: `android:actionBarStyle`  ==>  `android:titleTextStyle`  ==>  `android:textColor`
	  +  Action bar tabs: `android:actionBarTabTextStyle`  ==>  `android:textColor`
	  +  Action buttons: `android:actionMenuTextColor`
	+  support: 规则一样，去掉`android:`前缀（`android:textColor`不需要去掉前缀）
	+  示例
	```xml
	<?xml version="1.0" encoding="utf-8"?>
	<resources>
		<!-- the theme applied to the application or activity -->
		<style name="CustomActionBarTheme"
			parent="@style/Theme.AppCompat">
			<item name="android:actionBarStyle">@style/MyActionBar</item>
			<item name="android:actionBarTabTextStyle">@style/MyActionBarTabText</item>
			<item name="android:actionMenuTextColor">@color/actionbar_text</item>
	
			<!-- Support library compatibility -->
			<item name="actionBarStyle">@style/MyActionBar</item>
			<item name="actionBarTabTextStyle">@style/MyActionBarTabText</item>
			<item name="actionMenuTextColor">@color/actionbar_text</item>
		</style>
	
		<!-- ActionBar styles -->
		<style name="MyActionBar"
			parent="@style/Widget.AppCompat.ActionBar">
			<item name="android:titleTextStyle">@style/MyActionBarTitleText</item>
	
			<!-- Support library compatibility -->
			<item name="titleTextStyle">@style/MyActionBarTitleText</item>
		</style>
	
		<!-- ActionBar title text -->
		<style name="MyActionBarTitleText"
			parent="@style/TextAppearance.AppCompat.Widget.ActionBar.Title">
			<item name="android:textColor">@color/actionbar_text</item>
			<!-- The textColor property is backward compatible with the Support Library -->
		</style>
	
		<!-- ActionBar tabs text -->
		<style name="MyActionBarTabText"
			parent="@style/Widget.AppCompat.ActionBar.TabText">
			<item name="android:textColor">@color/actionbar_text</item>
			<!-- The textColor property is backward compatible with the Support Library -->
		</style>
	</resources>
	```
  +  Customize the Tab Indicator（图标/左侧按钮）
    +  standard: `android:actionBarTabStyle`  ==>  `android:background`
	+  示例
	```xml
	<?xml version="1.0" encoding="utf-8"?>
	<resources>
		<!-- the theme applied to the application or activity -->
		<style name="CustomActionBarTheme"
			parent="@style/Theme.AppCompat">
			<item name="android:actionBarTabStyle">@style/MyActionBarTabs</item>
	
			<!-- Support library compatibility -->
			<item name="actionBarTabStyle">@style/MyActionBarTabs</item>
		</style>
	
		<!-- ActionBar tabs styles -->
		<style name="MyActionBarTabs"
			parent="@style/Widget.AppCompat.ActionBar.TabView">
			<!-- tab indicator -->
			<item name="android:background">@drawable/actionbar_tab_indicator</item>
	
			<!-- Support library compatibility -->
			<item name="background">@drawable/actionbar_tab_indicator</item>
		</style>
	</resources>
	```
  +  [ActionBar全部style属性](http://developer.android.com/guide/topics/ui/actionbar.html#Style)
  +  [ActionBar Style Generator](http://jgilfelt.github.io/android-actionbarstylegenerator/)
+  Overlaying the Action Bar
  +  hide/show可以控制隐藏/显示，但是直接使用会导致Activity重新layout
  +  使用overlay mode可以避免重新layout，而使用带有透明度的ActionBar可以使得ActionBar底部的View也可见
  +  起用overlay mode
	```xml
	<resources>
		<!-- the theme applied to the application or activity -->
		<style name="CustomActionBarTheme"
			parent="@android:style/Theme.AppCompat">
			<item name="android:windowActionBarOverlay">true</item>
	
			<!-- Support library compatibility -->
			<item name="windowActionBarOverlay">true</item>
		</style>
	</resources>
	```
  +  设置root layout的顶部边距，使得其完全可见
    +  `android:paddingTop="?android:attr/actionBarSize"`
	+  `android:paddingTop="?attr/actionBarSize"`
+  多语言支持：创建`/values/strings.xml`，`/values-es/strings.xml`，`/values-fr/strings.xml`等资源文件夹，其中的资源定义为相同的id，则引用时会自动根据系统设置语言选择对应文件夹下的资源
+  多尺寸支持：`drawable-xhdpi`，`drawable-hdpi/`...等等尺寸相关的资源
+  多系统版本支持
  +  minSdkVersion，targetSdkVersion
  +  运行时检查：`Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB`
+  Fragment之间通信
  +  通过Activity间接实现，Activity实现接口，Fragment获取（`getActivity()`）、cast、调用
  +  EventBus
