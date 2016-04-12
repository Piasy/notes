# 《Android源码设计模式解析与实战》一书

## [面向对象的六大原则](../misc/OOP6Principles.md)

## 单例模式
+  建议实现方式
  +  无参数时，static inner holder class方式：
  
  ```java
  public class Singleton {
	  private Singleton() {
		  // singleton
	  }
	  
	  public static getInstance() {
		  return InstanceHolder.sInstance;
	  }
	  
	  private static class InstanceHolder {
		  private static final Singleton sInstance = new Singleton();
	  }
  }
  ```
  
  +  有参数时，优化的double check lock (DCL)方式，注意`volatile`关键字，在JDK1.5及以后，用于解决this指针逃逸问题：
  
  ```java
  public class Singleton {
      private static volatile Singleton sInstance;
    
      private final Context mContext;
    
	  private Singleton(Context context) {
		    mContext = context;
	  }
	  
	  public static getInstance(Context context) {
            if (sInstance == null) {
                synchronized(Singleton.class) {
                    if (sInstance == null) {
                        sInstance = new Singleton(context);
                    }
                }
            }
            
            return sInstance;
	  }
  }
  ```
  
  +  [this指针逃逸问题](http://blog.itpub.net/28912557/viewspace-762047/)
  
    在DCL单例实现中，`sInstance = new Singleton(context);`实际上大致会进行三个操作：
    
    1. 为`Singleton`的实例分配内存；
    2. 调用`Singleton`的构造函数，初始化成员变量；
    3. 把`sInstance`指向新分配的内存空间（此时`sInstance`就不是`null`了）；
    
    由于Java memory model的内存缓存模型（寄存器，cache，主存），以及允许指令重排，在缓存层面，上述三步中后两步顺序是无法保证的，有可能是`1-2-3`，也有可能是`1-3-2`。如果是`1-3-2`，在A线程执行，第三步执行完之后，切换到B线程，则B线程执行`getInstance`时将直接返回`sInstance`，因为B线程将直接命中缓存，但是它的成员变量仍未初始化，一旦使用就会出现问题。
    
    而`volatile`关键字则能保证每次读数据都从主存读取，不会受到缓存的影响，所以B线程从主存读到的仍是null，就不会出现问题了。
  
+  安卓系统的实践（部分）
  +  各种system service就是单例，虽然他们使用binder机制和真正的service跨进程通信，但是在本地的proxy就是单例的（APP进程中的单例）；
  +  每个APP都有各种各样的`Context`，它的实现类是`ComtextImpl`，它在static代码块中初始化各种system service，并保存在一个map中，后续获取的时候将直接从map中获取；android 23起，初始化system service、缓存、单例逻辑的代码从`ContextImpl`转移到了`SystemServiceRegistry`中；
  +  `LayoutInflater`工作原理
    +  实现类是`PhoneLayoutInflater`
    +  内置view，在xml定义时不用声明完整包名，因为`PhoneLayoutInflater`在`onCreateView`函数中会自动为其添加包名前缀
    +  最终view的创建都是通过`createView`函数完成，在其中通过反射创建view实例
    +  `inflate`函数会解析xml布局文件，深度优先遍历，逐层创建view（通过`createViewFromTag`函数），并最终形成一棵view的树
  
## Builder模式
+  [AutoValue](https://github.com/google/auto/tree/master/value)及其在Android平台的版本[AutoParcel](https://github.com/frankiesardo/auto-parcel)，不仅支持immutable，还支持builder模式
+  `WindowManager`
  +  实现类为`WindowManagerImpl`
  +  Activity, Fragment, Dialog等组建的view显示，都是通过的`Window::setContentView()`方法来实现的
  +  `Window`是抽象类，其实现类为`PhoneWindow`，**它的setContentView方法，怎么和WindowManager关联起来的？？**
  +  `WindowManager`添加、移除view的实现工作在`WindowManagerGlobal`类中
  +  `ViewRootImpl`是framework层与native层通信的桥梁，继承自`Handler`
  +  WMS只负责管理手机屏幕上View的z-order，即View的图层顺序，WMS管理的是属于某个window下的view

## 原型模式
+  在已有对象的基础上构造新对象，通常是clone；clone的效率通常比new的效率高，但不绝对；需要注意深浅拷贝的问题。
+  Intent的查找与匹配
  +  系统启动之后，`PackageManagerService`会扫描系统内所有的APP，解析其manifest文件，得到所有APP注册的所有各类组件，保存在系统中（内存）；后续使用Intent进行跳转时，通过查找保存的组件信息，得知应该启动哪个APP（组件）；
  +  显式Intent：直接指定了响应Activity；隐式Intent：只指定了Action；
  +  启动Activity时，会向PackageManagerService查询intent对应的activity，在`PackageManagerService::queryIntentActivities(...)`方法中；

## 工厂方法
+  用工厂去创建产品（对象），工厂和产品都可以提供抽象类，具体类，以达到实现解耦的目的
+  Activity的启动过程
  +  `ActivityThread::main()`是app的执行起点  ==>
  +  `thread.attach(false)`把ActivityThread绑定到ActivityManagerService，它调用了`ActivityManagerService::attachApplication`方法  ==>
  +  `ActivityManagerService::attachApplication`方法中间接调用了`ActivityThread.ApplicationThread::bindApplication`和`ActivityManagerService::attachApplicationLocked`，前者把ApplicationThread对象绑定到ActivityManagerService  ==>
  +  `ActivityManagerService::attachApplicationLocked`  ==>
  +  `ActivityManagerService::realStartActivityLocked`  ==>
  +  `ActivityThread.ApplicationThread::scheduleLaunchActivity`  ==>
  +  `ActivityThread.H::handleMessage`  ==>
  +  `ActivityThread::handleLaunchActivity`  ==>  
  +  `ActivityThread::performLaunchActivity`  ==>
  +  `Instrumentation::callActivityOnCreate`  ==>
  +  `Activity::performCreate`  ==>
  +  `Activity::onCreate`

## 抽象工厂
四种角色：抽象的工厂类，定义工厂的接口；具体的工厂类，实现生产产品；抽象的产品类，定义产品的接口；具体的产品类，实现产品的功能；

## 策略模式
+  某一需求可以有多种实现算法，将每种算法独立封装，且它们之间可以相互替换（实现相同的接口）。策略模式让算法独立于使用它们的客户端而独立变化。
+  安卓系统动画原理
  +  `view.startAnimation()`，动画是怎么实现的？随时间改变view属性的值；
  +  整个过程是怎样的？绘制时利用`TimeInterpolator`获取绘制的时间百分比，再利用`TypeEvaluator`把百分比计算为属性值，设置给view；
  +  怎么做到随时间流逝持续进行上述过程？
    +  `View::startAnimation(Animation animation)`在设置了animation之后，调用`invalidate`，invalidate最终调用到`ViewGroup::drawChild(Canvas canvas, View child, long drawingTime)`  ==>
    +  `View::draw(Canvas canvas, ViewGroup parent, long drawingTime)`  ==>
    +  `View::applyLegacyAnimation(ViewGroup parent, long drawingTime, Animation a, boolean scalingRequired)` ==>
    +  `applyLegacyAnimation`函数中，完成了动画初始化、动画操作、界面刷新（本次动画操作完成后，再次invalidate）
    +  动画操作在`Animator::getTransformation(long currentTime, Transformation outTransformation)`函数中实现
  +  `ValueAnimator`原理
    +  动画属性都保存在`PropertyValuesHolder`类中
    +  `ValueAnimator::start()`调用后，将会把动画指令发送给内部的`ValueAnimator.AnimationHandler`类，而handler则使用`Choreographer`来进行定时的刷新
    +  刷新时调用了`ValueAnimator::doAnimationFrame(long frameTime)`  ==>
    +  `ValueAnimator::animationFrame(long currentTime)`  ==>
    +  `ValueAnimator::animateValue(float fraction)`  ==>
    +  具体实现类（例如`ObjectAnimator`）的重载中，`ObjectAnimator::animateValue(float fraction)`  ==>
    +  `PropertyValuesHolder::setAnimatedValue(Object target)`，在其中通过反射，为view设置属性
  +  `ValueAnimator`的代码使用反射工作，设置动画属性时传入的是字符串，容易产生错误，有两个不错的库对这一点进行了优化：[ViewAnimator](https://github.com/florent37/ViewAnimator), [AnimatorCompat](https://github.com/zzz40500/AnimatorCompat)

## 状态模式
一个对象的行为取决于它的状态，最直接的实现是各个函数中对状态进行判断（if-else或switch），采取不同的行为。状态模式则是把状态抽象为一个类，不同状态下的行为封装为不同的状态实现类。改变对象的状态时，只需要修改其状态对象，即可达到修改其行为的目的。

## 安卓事件输入系统
+  `InputReader`，从硬件的事件（CPU中断）中，读取输入事件，并封装为`Event`对象，然后分发给`InputDispatcher`
+  `InputDispatcher`负责接收来自`InputReader`的事件，并分发给合适的窗口，同时监控ANR
+  `InputReaderThread`，`InputDispatcherThread`，`InputManager`，`InputManagerService`，`WindowManagerService`

## 观察者模式
又称订阅模式，定义了对象间一对多的依赖关系，当特定的对象（subject）变化时，所有依赖它的对象（observer）都会得到通知并自动更新。

+  ListView中的观察者模式
  +  更改adapter数据集之后，会调用`notifyDataSetChanged`
  +  会调用其内部的observer的`onChanged`方法
  +  内部的observer是AdapterView的内部类（非静态），其onChanged方法会触发AdapterView重新布局，达到刷新UI的目的
+  BroadcastReceiver中的观察者模式
  +  代码注册receiver过程：`registerReceiver` ==> `ContextWrapper::registerReceiver` ==>
  +  `ContextImpl::registerReceiver` ==> `ContextImpl::registerReceiverInternal`
    +  `LoadedApk::getReceiverDispatcher`函数创建了一个`IIntentReceiver`对象（Binder对象）
    +  `ActivityManagerNative.getDefault().registerReceiver`向ActivityManagerService注册receiver，并把`IIntentReceiver`传递进去，用于ActivityManagerService通知activity接收广播  ==>
  +  `ActivityManagerProxy::registerReceiver` ===>
  +  `ActivityManagerService::registerReceiver`
    +  sticky处理：如果是sticky intent，最后一次发送此广播的时候，ActivityManagerService会把这个intent保存起来，后面注册相同action的接收器时，就会得到最后一次的广播，并重放
    +  receiver会保存在ActivityManagerService中，并且会把receiver和filter进行关联
+  发送广播
  +  sendBroadcast会把广播通过binder发送给ActivityManagerService，后者通难过广播的action找到相应的接收器，然后把广播放进自己的消息队列中
  +  ActivityManagerService的消息队列循环中会处理这个广播，通过binder分发给注册的ReceiverDispatcher，后者把这个广播放到注册Activity所在线程的消息队列中
  +  ReceiverDispatcher的内部类Args在注册Activity所在线程的消息队列循环中处理这个广播，即调用`receiver.onReceive`
+  ActivityManagerService使用一个map保存了filter与receiver，注册就是加入map，发送就是查询map并完成分发
