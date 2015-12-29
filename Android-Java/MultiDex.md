# MultiDex专题

## 首先，了解自己当前的状况
+  dex方法数统计工具
  +  [Dexcount Gradle Plugin](https://github.com/KeepSafe/dexcount-gradle-plugin)，gradle插件，每次打包过程中把每个包的方法数写到build文件夹下的一个文件中
  +  [dex-method-counts](https://github.com/mihaip/dex-method-counts)，命令行工具，统计dex文件内的方法数
  +  [ClassyShark](https://github.com/google/android-classyshark)，GUI工具，查看apk内的dex分包，每个dex文件的方法数统计，每个dex文件里面有哪些class
  +  [ProGuard](ProGuard.md)，移除未引用的类，避免触及65535方法上限
  +  这些工具都可以用来了解当前工程的方法数（使用ProGuard前后），知悉各个包、各个库的方法数量，对于方法数很多、使用量少的库，应该移除依赖，或者通过适当配置ProGuard来移除，应该尽量避免MultiDex
  
## 尽量减少方法数，避免MultiDex，无法避免时优化MultiDex的性能
+  multidex时，放到主dex文件的类是由proguard来检测的，有时它并不是完整的，需要手动配置，还可以通过检测冷启动期间被加载了哪些类，然后通过配置把它们加入主dex中，从而达到multidex时冷启动加速的效果。[详情](https://medium.com/groupon-eng/android-s-multidex-slows-down-app-startup-d9f10b46770f)
+  全新的Android编译系统：[Jack & Jill](http://tools.android.com/tech-docs/jackandjill)（尚不成熟）

## 副dex异步加载技术
+  使用pre-dex jar来减小主dex文件的大小，但需要保证在使用某个库的类之前，pre-dex jar已经被加载，[详情](https://medium.com/@Macarse/lazy-loading-dex-files-d41f6f37df0e)
+  [美团、Facebook、微信团队对MultiDex加载的优化简述](http://zongwu233.github.io/the-touble-of-multidex/?)
