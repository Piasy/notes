# Best Practices for Performance

## Managing Your App's Memory
+  安卓系统的内存没有swap机制，但是有分页机制
+  安卓系统不会整理堆上的内存，即：小块的可用内存可能无法再被使用
+  `ActivityManager::getMemoryClass`返回系统对堆内存大小的限制
+  谨慎使用Service
+  用户界面销毁时，释放内存资源
+  内存吃紧时（`onTrimMemory()`回调），释放非必须的内存资源
+  检查允许使用的堆大小：`getMemoryClass()`, `getLargeMemoryClass()`
+  bitmap要小心使用
+  使用优化的容器类型：`SparseArray`, `SparseBooleanArray`, `LongSparseArray`
+  注意数据结构的内存overhead
+  不要过度使用继承
+  合理选择序列化/反序列化方式
+  合理选择依赖注入框架，dagger比较不错，但是guice和roboguice开销很大
+  合理选择第三方库
+  ProGuard, zipalign
+  Service可以考虑使用跨进程

## 优化layout
+  不要简单地认为RelativeLayout慢，需要测量！也许两层LinearLayout比一层RelativeLayout更慢，但也有可能更快，都需要用测量来确认；
+  include和merge标签的使用，merge作为layout的根节点，在include的时候将移除，直接将子节点展开到它的父节点中；
+  使用ViewStub按需加载layout，ViewStub的加载很轻量，在需要的时候再加载它所stub的layout；ViewStub与merge无法同时使用；
