#Effective Java一书笔记

##对象的创建与销毁
+  Item 1: 使用static工厂方法，而不是构造函数创建对象  
仅仅是创建对象的方法，并非Factory Pattern
  +  优点
    +  命名、接口理解更高效，通过工厂方法的函数名，而不是参数列表来表达其语义
	+  Instance control，并非每次调用都会创建新对象，可以使用预先创建好的对象，或者做对象缓存；便于实现单例；或不可实例化的类；对于immutable的对象来说，使得用`==`判等符合语义，且更高效；
	+  工厂方法能够返回任何返回类型的子类对象，甚至是私有实现；使得开发模块之间通过接口耦合，降低耦合度；而接口的实现也将更加灵活；接口不能有static方法，通常做法是为其再创建一个工厂方法类，如Collection与Collections；
	+  Read More: Service Provider Framework
  +  缺点
    +  仅有static工厂方法，没有public/protected构造函数的类将无法被继承；见仁见智，这一方面也迫使开发者倾向于组合而非继承；
	+  Javadoc中不能和其他static方法区分开，没有构造函数的集中显示优点；但可以通过公约的命名规则来改善；
  +  小结  
  static工厂方法和public构造函数均有其优缺点，在编码过程中，可以先考虑一下工厂方法是否合适，再进行选择。
+  Item 2: 使用当构造函数的参数较多，尤其是其中还有部分是可选参数时，使用Builder模式
  +  以往的方法
    +  Telescoping constructor：针对可选参数，从0个到最多个，依次编写一个构造函数，它们按照参数数量由少到多逐层调用，最终调用到完整参数的构造函数；代码冗余，有时还得传递无意义参数，而且容易导致使用过程中出隐蔽的bug；
    +  JavaBeans Pattern：灵活，但是缺乏安全性，有状态不一致问题，线程安全问题；
  +  Builder Pattern
    +  代码灵活简洁；具备安全性；
    +  immutable
    +  参数检查：最好放在要build的对象的构造函数中，而非builder的构建过程中
    +  支持多个field以varargs的方式设置（每个函数只能有一个varargs）
    +  一个builder可以build多个对象
    +  Builder结合泛型，实现Abstract Factory Pattern
    +  传统的抽象工厂模式，是用Class类实现的，然而其有缺点：newInstance调用总是去调用无参数构造函数，不能保证存在；newInstance方法会抛出所有无参数构造函数中的异常，而且不会被编译期的异常检查机制覆盖；可能会导致运行时异常，而非编译期错误；
  +  小结  
  Builder模式在简单地类（参数较少，例如4个以下）中，优势并不明显，但是需要予以考虑，尤其是当参数可能会变多时，有可选参数时更是如此。
+  Item 3: 单例模式！  
不管以哪种形式实现单例模式，它们的核心原理都是将构造函数私有化，并且通过静态方法获取一个唯一的实例，在这个获取的过程中你必须保证线程安全、反序列化导致重新生成实例对象等问题，该模式简单，但使用率较高。
  +  double-check-locking  
    ```java
        private static volatile RestAdapter sRestAdapter = null;
        public static RestAdapter provideRestAdapter() {
            if (sRestAdapter == null) {
                synchronized (RestProvider.class) {
                    if (sRestAdapter == null) {
                        sRestAdapter = new RestAdapter();
                    }
                }
            }
    
            return sRestAdapter;
        }
    ```
  DCL可能会失效，因为指令重排可能导致同步解除后，对象初始化不完全就被其他线程获取；使用volatile关键字修饰对象，或者使用static SingletonHolder来避免该问题（后者JLS推荐）；
  +  class的static代码：一个类只有在被使用时才会初始化，而类初始化过程是非并行的，这些都由JLS能保证
  +  用enum实现单例
  +  还存在反射安全性问题：利用反射，可以访问私有方法，可通过加一个控制变量，该变量在getInstance函数中设置，如果不是从getInstance调用构造函数，则抛出异常；
+  Item 4: 将构造函数私有化，使得不能从类外创建实例，同时也能禁止类被继承  
  util类可能不希望被实例化，有其需求
+  Item 5: 避免创建不必要的对象
  +  提高性能：创建对象需要时间、空间，“重量级”对象尤甚；immutable的对象也应该避免重复创建，例如String；
  +  避免auto-boxing
  +  但是因此而故意不创建必要的对象是错误的，使用object pool通常也是没必要的
  +  lazy initialize也不是特别必要，除非使用场景很少且很重量级
  +  Map#keySet方法，每次调用返回的是同一个Set对象，如果修改了返回的set，其他使用的代码可能会产生bug
  +  需要defensive copying的时候，如果没有创建一个新对象，将导致很隐藏的Bug
+  Item 6: 不再使用的对象一定要解除引用，避免memory leak
  +  例如，用数组实现一个栈，pop的时候，如果仅仅是移动下标，没有把pop出栈的数组位置引用解除，将发生内存泄漏
  +  程序发生错误之后，应该尽快把错误抛出，而不是以错误的状态继续运行，否则可能导致更大的问题
  +  通过把变量（引用）置为null不是最好的实现方式，只有在极端情况下才需要这样；好的办法是通过作用域来使得变量的引用过期，所以尽量缩小变量的作用域是很好的实践；注意，在Dalvik虚拟机中，存在一个细微的bug，可能会导致内存泄漏，[详见](MemoryLeak.md)
  +  当一个类管理了一块内存，用于保存其他对象（数据）时，例如用数组实现的栈，底层通过一个数组来管理数据，但是数组的大小不等于有效数据的大小，GC器却并不知道这件事，所以这时候，需要对其管理的数据对象进行null解引用
  +  当一个类管理了一块内存，用于保存其他对象（数据）时，程序员应该保持高度警惕，避免出现内存泄漏，一旦数据无效之后，需要立即解除引用
  +  实现缓存的时候也很容易导致内存泄漏，放进缓存的对象一定要有换出机制，或者通过弱引用来进行引用
  +  listner和callback也有可能导致内存泄漏，最好使用弱引用来进行引用，使得其可以被GC