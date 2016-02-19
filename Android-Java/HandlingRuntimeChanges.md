# Handling Runtime Changes
Runtime Changes包括orientation，键盘可见性，语言设置等，这些内容发生变化后，
系统将重启Activity（先执行`onDestroy`，再执行`onCreate`），以便APP可以响应
这些变化。

`onSaveInstanceState()`和`onRestoreInstanceState()`回调在这种情形下可以
使得APP能保存已有状态，在Activity重新创建时能够恢复已有状态，为用户提供一致的体验。

如果需要把已加载的数据应用到新的状态中，有两种方式：

+  在配置发生变化时，保留数据
+  当配置发生变化时，不要重新创建Activity（系统默认行为），而是响应相应事件，
手动改变Activity状态

## Retaining an Object During a Configuration Change
`onSaveInstanceState()`和`onRestoreInstanceState()`并不是设计用来传递大量数据的，
其传递的Bundle对象有大小限制。而且Bundle中的数据都会先反序列化再序列化，耗时较多。

可以使用Fragment，调用`Fragment::setRetainInstance(true)`，在Activity重新创建之后，
通过FragmentManager获取到重用的Fragment对象，进而获取到已有的数据。这里需要注意的是，
Fragment不能直接或者间接持有Activity的引用，否则可能会导致老的Activity对象的内存泄漏。

利用这一方式，有一种`HeadlessFragment`的用法，这个Fragment没有UI，只负责后台加载数据，
它不会因为Activity的配置变化销毁而销毁，可以保证数据获取过程的连续性。

## Handling the Configuration Change Yourself
Activity可以在manifest中声明自行处理配置变化，同时`onConfigurationChanged()`回调会
在配置变化时被执行。这种方式是最为复杂的，应该是最后的考虑选项。

```xml
android:configChanges="orientation|screenSize|keyboardHidden"
```

配置发生变化之后，`getResources()`函数返回的对象是新配置下的资源对象，可以直接使用，

## Loaders
相比于上述通过Fragment保留数据的方式，更建议使用Loaders进行替换。Loaders自安卓3.0引入，
用于异步加载数据，Activity和Fragment均可以使用。

Loader框架包含了一个`CursorLoader`实现，包括了异步请求数据，Activity/Fragment销毁重新
创建时直接返回已有数据等功能。

+  Activity/Fragment重新创建时数据保留：Loader对象并不会被销毁，其生命周期由framework维护
（包括不会销毁与需要销毁的情形）；
+  异步加载：基于AsyncTask，但异步逻辑无需实现；
+  数据更新：自行检测数据变更，但是提供了更新数据的相应API；
+  framework/support内的实现，Google背书；
+  framework提供了[`LoaderTestCase`](http://developer.android.com/reference/
android/test/LoaderTestCase.html)类，用于测试；
+  需要什么测试？
  +  Loader有其数据更新/发送/重置的逻辑，需要单元测试；
  +  Loader对于framework的依赖无法隔离（Handler, AsyncTask等），而且LoaderTestCase
  也是需要`AndroidJUnitTestRunner`执行的；
  +  Loader和数据源有对接，需要集成测试；
+  对比MVP？
  +  相当于presenter；
  +  能否结合？如果仅仅是对于设计模式来说，当然是可以的；但对[Mosby](https://github.com/
  sockeqwe/mosby)来说，两者是有重复工作的，例如retain state，life cycle绑定，而且这部分
  工作都能满足需求，也有背书；
