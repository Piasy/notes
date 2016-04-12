# 深入Android frameworks
《深入理解Android 5源代码》一书的内容实在太渣，转战博客系列：[老罗的Android之旅](http://blog.csdn.net/Luoshengyang/article/)

## Java Native Interface(JNI)系统
+  JNI函数注册：把Java中声明的native方法与C/C++代码一一对应起来
  +  `JNINativeMethod`结构体，name, signature, fnPtr三个成员
  +  signature部分：V - void - void, Z - jboolean - boolean, J - jlong - long, 
  S - jshort - short, Ljava/lang/String; - jstring - String, 
  Ljava/net/Socket; - jobject - Socket
  +  通过在C/C++代码中设置好Java native方法对应的结构体数据，就可以达到将其对应起来的目的
  +  所有的注册工作都在`frameworks/base/core/jni/AndroidRuntime.cpp`中完成
  +  `AndroidRuntime::registerNativeMethods`函数调用了`JNIHelp.cpp`中的
  `jniRegisterNativeMethods`方法，=> `C_JNIEnv::RegisterNatives`
  +  如果不通过上述方式注册native函数，函数被调用时AndroidRuntime会从所有已载入的so库中
  搜索被调用的函数，效率较低，而上述注册会加速native函数的查找，只需在JNINativeMethod数组
  中查找即可
  +  而且由于上述注册机制，使得可以在运行时动态注册/hook
  +  动态注册
    +  Java层调用`System.loadLibrary`加载so库
    +  查找so库中的`JNI_OnLoad`函数，如果存在，则执行之
    +  可以在JNI_OnLoad函数中进行动态注册
  
## Hardware Abstract Layer(HAL)
+  将硬件抽象化，为操作系统提供虚拟硬件平台，操作系统的代码和硬件驱动的代码完全分离，软硬件完全分离，
便于保护硬件厂商的知识产权，也便于软硬件测试并行化
+  主要包括`hw_module_t`, `hw_module_methods_t`, `hw_device_t`
+  `hw_module_methods_t`为所有硬件操作定义了统一的API
+  硬件模块的编写有一定规范，HAL按照该规范载入硬件模块的SO库，获取绑定其中的符号、代码

## Binder机制(跨进程通信IPC)
+  [Android深入浅出之Binder机制](http://www.cnblogs.com/innost/archive/2011/01/09/1931456.html)
  +  系统服务的初始化流程
    +  每个使用Binder机制进行IPC的进程，都需要打开一次`/dev/binder`设备，并进行内存映射，方便后续操作：`sp<ProcessState> proc(ProcessState::self())`；
    +  每个进程也需要创建一个BpBinder对象，并将其转换为IServiceManager类型，保存到进程数据结构ProcessState中：`sp<IServiceManager> sm = defaultServiceManager()`；
    +  其实上面的`proc`和`sm`变量都没有使用，重要的工作是对两个函数的调用，而之所以要定义未使用的变量，是为了利用对象的析构机制自动释放资源，如果未定义变量，那返回的对象会立即析构，而如果定义了变量，则会在离开作用域时析构；
    +  初始化相应的服务：`MediaPlayerService::instantiate()`；
    +  创建线程池，加入线程池，并在其中开始循环，以便响应通过Binder发送的IPC请求：`ProcessState::self()->startThreadPool()`，`IPCThreadState::self()->joinThreadPool()`；
  +  IBinder, BBinder, BpBinder, IInterface, BpInterface, IServiceManager, BpServiceManager, 
  MediaPlayerService, BnMediaPlayerService 它们之间什么关系？
    +  BBinder : IBinder : RefBase，定义了binder的API：`transact(...)`等基础API；
    +  BpRefBase : RefBase，定义了基础的代理API：`remote()`；
    +  BpBinder : IBinder，它也定义了代理的API：`remoteBinder()`；
    +  IInterface : RefBase，定义了所有的service的公共API：`asBinder(...)`；
    +  BnInterface : INTERFACE, BBinder，它是个模板类，用法是`BnInterface<IXXX>`，这样声明的类型就会同时继承IXXX和BBinder，`BnInterface<IXXX>`是IXXX的实现者，提供API：`queryLocalInterface(...)`；
    +  BpInterface : INTERFACE, BpRefBase，它是个模板类，用法是`BpInterface<IXXX>`，这样声明的类型就会同时继承IXXX和BpRefBase，`BpInterface<IXXX>`是IXXX的代理服务提供者（代理）；
    +  IServiceManager : IInterface，service manager也是一个服务，只不过它比较特殊，它的功能是管理其他所有的服务，其他所有的服务都需要把自己注册到service manager的服务列表中；
    +  `BnServiceManager : BnInterface<IServiceManager>`，是IServiceManager的实现者；
    +  `BpServiceManager : BpInterface<IServiceManager>`，是IServiceManager的代理；
    +  `ServiceManager.java`，是Java层对ServiceManager的定义，它通过`ServiceManagerNative.asInterface(BinderInternal.getContextObject())`获得`IServiceManager`实例，而该方法则是通过查询native层service manager的服务列表获取服务实例；
    +  IMediaPlayerService: IInterface，定义了MediaPlayerService的API，包括远程调用，属性访问；
    +  `BnMediaPlayerService: BnInterface<IMediaPlayerService>`，是IMediaPlayerService的实现者；
    +  `BpMediaPlayerService: BpInterface<IMediaPlayerService>`，是IMediaPlayerService的代理；
    
    +  IXXX是接口的定义，实现者是BnXXX，使用者通过代理BpXXX和接口（服务）打交道；
    +  Bp ==> Binder proxy；
    +  BpServiceManager可以理解为IServiceManager的代理，对用户透明，是仅供内部使用的API；
    +  Bn ==> Binder native；
    +  Bn和Bp相对，Bp是代理，则Bn就是和代理通信的另一端（实现端），Bp通过和Bn通信，代理IXXX向使用者提供服务；
  +  addService的IPC过程
    +  调用的是`remote()->transact(...)`函数，而remote返回的实际上就是初始化ServiceManager时创建的BpBinder；
    +  ==> `IPCThreadState::self()->transact(...)`，在其中通过binder设备发送命令和数据，同时读取返回的命令和数据；
  +  自行实现一整套service
    +  如果是纯native的service，没有应用层入口，则需要和MediaPlayerService一样，实现一个main函数，并在其他同类service的启动处启动自定义的service；
    +  定义service接口IXXX : IInterface，定义需要的API；
    +  定义BnXXX和BpXXX，完成service的实现和代理；
+  [浅谈Service Manager成为Android进程间通信（IPC）机制Binder守护进程之路](http://blog.csdn.net/luoshengyang/article/details/6621566)
  +  binder机制为service manager预留了通道，使其可以率先把自己注册为binder守护进程
  +  binder IPC机制client和server通信只需要一次数据拷贝：client从用户空间拷贝到内核空间；server和内核空间（binder驱动）共享数据
  +  通过ioctl机制通知binder驱动自己成为守护进程之后，service manager进程将进入binder_loop，循环等待client的请求到来
+  [浅谈Android系统进程间通信（IPC）机制Binder中的Server和Client获得Service Manager接口之路](http://blog.csdn.net/luoshengyang/article/details/6627260)
  +  `sp<IServiceManager> defaultServiceManager();`这一方法实现了单例模式，单例保存在`gDefaultServiceManager`中
  +  创建单例：`gDefaultServiceManager = interface_cast<IServiceManager>(ProcessState::self()->getContextObject(NULL));`
  +  简化为：`gDefaultServiceManager = new BpServiceManager(new BpBinder(0));`

## [Android启动运行在独立进程的Service](http://blog.csdn.net/luoshengyang/article/details/6677029)
+  跨进程，当然就会使用binder机制
+  包括三次跨进程通信
  +  从主进程（调用startService的进程）调用到ActivityManagerService进程中，完成新进程的创建，新进程开始执行
  +  从新进程调用到ActivityManagerService进程中，新进程告知ActivityManagerService自己已经准备就绪
  +  从ActivityManagerService进程又回到新进程中，传递了相关信息，最终将服务启动起来

## [Android应用程序的Activity启动过程](http://blog.csdn.net/luoshengyang/article/details/6685853)
+  step2: Activity.startActivity => 
+  step3: Activity.startActivityForResult
+  step4: Instrumentation.execStartActivity => 
+  step5: ActivityManagerProxy.startActivity == binder ==> 
+  step6: ActivityManagerService.startActivity => 
+  step6: ActivityManagerService.startActivityAsUser => 
+  step7: ActivityStackSupervisor.startActivityMayWait =>
+  step8: ActivityStackSupervisor.startActivityLocked =>
+  step9: ActivityStackSupervisor.startActivityUncheckedLocked =>
+  step10: ActivityStack.resumeTopActivityLocked =>  （此函数有两次调用，不同参数执行路径不同）
+  step11: ActivityStack.startPausingLocked =>
+  step12: ApplicationThreadProxy.schedulePauseActivity == binder ==>
+  step13: ActivityThread$ApplicationThread.schedulePauseActivity =>
+  step14: ActivityThread.queueOrSendMessage =>  (PAUSE_ACTIVITY)
+  step15: ActivityThread$H.handleMessage =>
+  step16: ActivityThread.handlePauseActivity =>  （在此处调用前一activity的onPause回调）
+  step17: ActivityManagerProxy.activityPaused == binder ==>
+  step18: ActivityManagerService.activityPaused =>
+  step19: ActivityStack.activityPaused =>
+  step20: ActivityStack.completePauseLocked =>
+  step21: ActivityStack.resumeTopActivityLocked =>
+  step22: ActivityStack.startSpecificActivityLocked =>  （如果对应进程不存在，则创建进程）
+  step23: ActivityManagerService.startProcessLocked =>
+  step24: ActivityThread.main =>
+  step25: ActivityManagerProxy.attachApplication == binder ==>
+  step26: ActivityManagerService.attachApplication =>
+  step27: ActivityManagerService.attachApplicationLocked =>
+  step28: ActivityStack.realStartActivityLocked =>
+  step29: ApplicationThreadProxy.scheduleLaunchActivity == binder ==>
+  step30: ApplicationThread.scheduleLaunchActivity =>
+  step31: ActivityThread.queueOrSendMessage =>  (LAUNCH_ACTIVITY)
+  step32: ActivityThread$H.handleMessage =>
+  step33: ActivityThread.handleLaunchActivity =>
+  step34: ActivityThread.performLaunchActivity =>  （此处通过反射创建新activity对象，然后通过mInstrumentation调用新activity的onCreate/onResume等回调）

+  Step1 - Step 11：Launcher通过Binder进程间通信机制通知ActivityManagerService，它要启动一个Activity；
+  Step 12 - Step 16：ActivityManagerService通过Binder进程间通信机制通知Launcher进入Paused状态；
+  Step 17 - Step 24：Launcher通过Binder进程间通信机制通知ActivityManagerService，它已经准备就绪进入Paused状态，于是ActivityManagerService就创建一个新的进程，用来启动一个ActivityThread实例（**how?**），即将要启动的Activity就是在这个ActivityThread实例中运行；
+  Step 25 - Step 27：ActivityThread通过Binder进程间通信机制将一个ApplicationThread类型的Binder对象传递给ActivityManagerService，以便以后ActivityManagerService能够通过这个Binder对象和它进行通信；
+  Step 28 - Step 35：ActivityManagerService通过Binder进程间通信机制通知ActivityThread，现在一切准备就绪，它可以真正执行Activity的启动操作了。
