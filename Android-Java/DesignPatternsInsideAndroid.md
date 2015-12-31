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
  
  +  this指针逃逸问题
  
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
  