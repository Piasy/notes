# Effective Java一书笔记

## 对象的创建与销毁
+  Item 1: 使用static工厂方法，而不是构造函数创建对象：仅仅是创建对象的方法，并非Factory Pattern
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
+  Item 7: 不要使用finalize方法
  +  finalize方法不同于C++的析构函数，不是用来释放资源的好地方
  +  finalize方法执行并不及时，其执行线程优先级很低，而当对象unreachable之后，需要执行finalize方法之后才能释放，所以会导致对象生存周期变长，甚至根本不会释放
  +  finalize方法的执行并不保证执行成功/完成
  +  使用finalize时，性能会严重下降
  +  finalize存在的意义
    +  充当“safety net”的角色，避免对象的使用者忘记调用显式termination方法，尽管finalize方法的执行时间没有保证，但是晚释放资源好过不释放资源；此处输出log警告有利于排查bug
    +  用于释放native peer，但是当native peer持有必须要释放的资源时，应该定义显式termination方法
  +  子类finalize方法并不会自动调用父类finalize方法（和构造函数不同），为了避免子类不手动调用父类的finalize方法导致父类的资源未被释放，当需要使用finalize时，使用finalizer guardian比较好：
    +  定义一个私有的匿名Object子类对象，重写其finalize方法，在其中进行父类要做的工作
    +  因为当父类对象被回收时，finalizer guardian也会被回收，它的finalize方法就一定会被触发

## Object的方法
尽管Object不是抽象类，但是其定义的非final方法设计的时候都是希望被重写的，finalize除外。
+  Item 8: 当重写equals方法时，遵循其语义
  +  能不重写equals时就不要重写
    +  当对象表达的不是值，而是可变的状态时
    +  对象不需要使用判等时
    +  父类已重写，且满足子类语义
  +  当需要判等，且继承实现无法满足语义时，需要重写（通常是“value class”，或immutable对象）
  +  当用作map的key时
  +  重写equals时需要遵循的语义
    +  Reflexive（自反性）: x.equals(x)必须返回true（x不为null）
    +  Symmetric（对称性）: x.equals(y) == y.equals(x)
    +  Transitive（传递性）: x.equals(y) && y.equals(z) ==> x.equals(z)
    +  Consistent（一致性）: 当对象未发生改变时，多次调用应该返回同一结果
    +  x.equals(null)必须返回false
  +  实现建议
    +  先用==检查是否引用同一对象，提高性能
    +  用instanceof再检查是否同一类型
    +  再强制转换为正确的类型
    +  再对各个域进行equals检查，遵循同样的规则
    +  确认其语义正确，编写测例
    +  重写equals时，同时也重写hashCode
    +  ！重写equals方法，传入的参数是Object
+  Item 9: 重写equals时也重写hashCode函数
  +  避免在基于hash的集合中使用时出错
  +  语义
    +  一致性
    +  当两个对象equals返回true时，hashCode方法的返回值也要相同
  +  hashCode的计算方式
    +  要求：equals的两个对象hashCode一样，但是不equals的对象hashCode不一样
    +  取一个素数，例如17，result = 17
    +  对每一个关心的field（在equals中参与判断的field），记为f，将其转换为一个int，记为c
      +  boolean: f ? 1 : 0
      +  byte/char/short/int: (int) f
      +  long: (int) (f ^ (f >> 32))
      +  float: Float.floatToIntBits(f)
      +  double: Double.doubleToLongBits(f)，再按照long处理
      +  Object: f == null ? 0 : f.hashCode()
      +  array: 先计算每个元素的hashCode，再按照int处理
    +  对每个field计算的c，result = 31 * result + c
    +  返回result
    +  编写测例
  +  计算hashCode时，不重要的field（未参与equals判断）不要参与计算
+  Item 10: 重写toString()方法
  +  增加可读性，简洁、可读、具有信息量
+  Item 11: 慎重重写clone方法
  +  Cloneable接口是一个mixin interface，用于表明一个对象可以被clone
  +  Contract
    +  x.clone() != x
    +  x.clone().getClass() ==  x.getClass()：要求太弱，当一个非final类重写clone方法的时候，创建的对象一定要通过super.clone()来获得，所有父类都遵循同样的原则，如此最终通过Object.clone()创建对象，能保证创建的是正确的类实例。而这一点很难保证。
    +  x.clone().equals(x)
    +  不调用构造函数：要求太强，一般都会在clone函数里面调用
  +  对于成员变量都是primitive type的类，直接调用super.clone()，然后cast为自己的类型即可（重写时允许返回被重写类返回类型的子类，便于使用方，不必每次cast）
  +  成员变量包含对象（包括primitive type数组），可以通过递归调用成员的clone方法并赋值来实现
  +  然而上述方式违背了final的使用协议，final成员不允许再次赋值，然而clone方法里面必须要对其赋值，则无法使用final保证不可变性了
  +  递归调用成员的clone方法也会存在性能问题，对HashTable递归调用深拷贝也可能导致StackOverFlow（可以通过遍历添加来避免）
  +  优雅的方式是通过super.clone()创建对象，然后为成员变量设置相同的值，而不是简单地递归调用成员的clone方法
  +  和构造函数一样，在clone的过程中，不能调用non final的方法，如果调用虚函数，那么该函数会优先执行，而此时被clone的对象状态还未完成clone/construct，会导致corruption。因此上一条中提及的“设置相同的值”所调用的方法，要是final或者private。
  +  重载类的clone方法可以省略异常表的定义，如果重写时把可见性改为public，则应该省略，便于使用；如果设计为应该被继承，则应该重写得和Object的一样，且不应该实现Cloneable接口；多线程问题也需要考虑；
  +  要实现clone方法的类，都应该实现Cloneable接口，同时把clone方法可见性设为public，返回类型为自己，应该调用super.clone()来创建对象，然后手动设置每个域的值
  +  clone方法太过复杂，如果不实现Cloneable接口，也可以通过别的方式实现copy功能，或者不提供copy功能，immutable提供copy功能是无意义的
  +  提供拷贝构造函数，或者拷贝工厂方法，而且此种方法更加推荐，但也有其不足
  +  设计用来被继承的类时，如果不实现一个正确高效的clone重写，那么其子类也将无法实现正确高效的clone功能
+  Item 12: 当对象自然有序时，实现Comparable接口
  +  实现Comparable接口可以利用其有序性特点，提高集合使用/搜索/排序的性能
  +  Contact
    +  sgn(x.compareTo(y)) == - sgn(y.compareTo(x))，当类型不对时，应该抛出ClassCastException，抛出异常的行为应该是一致的
    +  transitive: x.compareTo(y) > 0 && y.compareTo(z) > 0 ==> x.compareTo(z) > 0
    +  x.compareTo(y) == 0 ==> sgn(x.compareTo(z)) == sgn(y.compareTo(z))
    +  建议，但非必须：与equals保持一致，即 x.compareTo(y) == 0 ==> x.equals(y)，如果不一致，需要在文档中明确指出
  +  TreeSet, TreeMap等使用的就是有序保存，而HashSet, HashMap则是通过equals + hashCode保存
  +  当要为一个实现了Comparable接口的类增加成员变量时，不要通过继承来实现，而是使用组合，并提供原有对象的访问方法，以保持对Contract的遵循
  +  实现细节
    +  优先比较重要的域
    +  谨慎使用返回差值的方式，有可能会溢出

## Classes and Interfaces
+  Item 13: 最小化类、成员的可见性
  +  封装（隐藏）：公开的接口需要暴露，而接口的实现则需要隐藏，使得接口与实现解耦，降低模块耦合度，增加可测试性、稳定性、可维护性、可优化性、可修改性
  +  如果一个类只对一个类可见，则应该将其定义为私有的内部类，而没必要public的类都应该定义为package private
  +  为了便于测试，可以适当放松可见性，但也只应该改为package private，不能更高
  +  成员不能是非private的，尤其是可变的对象。一旦外部可访问，将失去对其内容的控制能力，而且会有多线程问题
  +  暴露的常量也不能是可变的对象，否则public static final也将失去其意义，final成员无法改变其指向，但其指向的对象却是可变的（immutable的对象除外），长度非0的数组同样也是有问题的，可以考虑每次访问时创建拷贝，或者使用`Collections.unmodifiableList(Arrays.asList(arr))`
+  Item 14: public class中，使用accessor method而非public field
  +  后者外部可以直接访问，失去了安全性
  +  package private或者private则可以不必这样
  +  把immutable的field置为public勉强可以接受，mutable的成员一定不能置为public
+  Item 15: 最小化可变性
  +  不提供可以改变本对象状态的方法
  +  保证类不可被继承
  +  使用final field
  +  使用private field
  +  在构造函数、accessor中，对mutable field使用defensive copy
  +  实现建议
    +  操作函数，例如BigInteger的add方法，不是static的，但也不能改变本对象的状态，则使用functional的方式，返回一个新的对象，其状态是本对象修改之后的状态
    +  如此实现的immutable对象生来就是线程安全的，无需同步操作，但应该鼓励共用实例，避免创建过多重复的对象
    +  正确实现的immutable对象也不需要clone, copy方法；可以适当引入Object cache；
  +  劣势
    +  每一个值都需要一个对象，调用改变状态的方法而创建一个新的对象，尤其是它是重量级的，开销会变大；连续调用这样的方法，影响更大；
    +  为常用的多次操作组合提供一个方法
  +  其他
    +  保证class无法被继承，除了声明为final外，还可以将默认构造函数声明为private或package private，然后提供public static工厂方法
    +  使用public static工厂方法，具体实现类可以有多个，还能进行object cache
    +  当实现Serializable接口是，一定要实现readObject/readResolve方法，或者使用ObjectOutputStream.writeUnshared/ObjectInputStream.readUnshared
  +  小结
    +  除非有很好的理由让一个Class mutable，否则应该使其immutable
    +  如果非要mutable，也应尽可能限制其可变性
+  Item 16: Favor composition (and forwarding) over inheritance
  +  跨包继承、继承不是被设计为应该被继承的实现类，是一件很危险的事情，继承接口、继承抽象类，当然是没问题的
  +  如果子类的功能依赖于父类的实现细节，那么一旦父类发生变化，子类将有可能出现Bug，即便代码都没有修改；而设计为应被继承的类，在修改后，是应该有文档说明的，子类开发者既可以得知，也可以知道如何修改
  +  例子：统计HashSet添加元素的次数
    +  用继承方式，重写add，addAll，在其中计数，这就不对，因为HashSet内部的addAll是通过调用add实现的
    +  但是通过不重写addAll也只不对的，以后有可能HashSet的实现就变了
    +  在重写中重新实现一遍父类的逻辑也是行不通的，因为这可能会导致性能问题、bug等，而且有些功能不访问私有成员也是无法实现的
    +  还有一个原因就是父类的实现中，可能会增加方法，改变其行为，而这一点，在子类中是无法控制的
  +  而通过组合的方式，将不会有这些问题，把另一个类的对象声明为私有成员，外部将无法访问它，自己也能在转发（forwarding）过程中执行拦截操作，也不必依赖其实现细节，这种组合、转发的实现被称为wrapper，或者Decorator pattern，或者delegation（严格来说不是代理，代理一般wrapper对象都需要把自己传入到被wrap的对象方法中？）
  +  缺点
    +  不适用于callback frameworks？
  +  继承应该在is-a的场景中使用
  +  继承除了会继承父类的API功能，也会继承父类的设计缺陷，而组合则可以隐藏成员类的设计缺陷
+  Item 17: Design and document for inheritance or else prohibit it
  +  一个类必须在文档中说明，每个可重写的方法，在该类的实现中的哪些地方会被调用（the class must document its self-use of overridable methods）。调用时机、顺序、结果产生的影响，包括多线程、初始化等情况。
  +  被继承类应该通过谨慎选择protected的方法或成员，来提供一些hook，用于改变其内部的行为，例如java.util.AbstractList::removeRange。
  +  The only way to test a class designed for inheritance is to write subclasses. 用于判断是否需要增加或者减少protected成员/方法，通常写3个子类就差不多了。
  +  You must test your class by writing subclasses before you release it.
  +  Constructors must not invoke overridable methods. 父类的构造函数比子类的构造函数先执行，而如果父类构造函数中调用了可重写的方法，那么就会导致子类的重写方法比子类的构造函数先执行，会导致corruption。
  +  如果实现了Serializable/Cloneable接口，neither clone nor readObject may invoke an overridable method, directly or indirectly. 重写方法会在deserialized/fix the clone’s state之前执行。
  +  如果实现了Serializable接口，readResolve/writeReplace必须是protected，而非private
  +  designing a class for inheritance places substantial limitations on the class.
  +  The best solution to this problem is to prohibit subclassing in classes that are not designed and documented to be safely subclassed. 声明为final class或者把构造函数私有化（提供public static工厂方法）。
  +  如果确实想要允许继承，就应该为每个被自己使用的可重写方法都写好文档
+  Item 18: Prefer interfaces to abstract classes
  +  Java类只允许单继承，接口可以多继承，使用接口定义类型，使得class hierarchy更加灵活
  +  定义mixin（optional functionality to be "mixed in"）时使用interface是很方便的，需要增加此功能的类只需要implement该接口即可，而如果使用抽象类，则无法增加一个extends语句
  +  接口允许构建没有hierarchy的类型系统
  +  使用接口定义类型，可以使得item 16中提到的wrapper模式更加安全、强大，
  +  skeletal implementation：该类为abstract，把必须由client实现的方法设为abstract，可以有默认实现的则提供默认实现
  +  simulated multiple inheritance：通过实现定义的接口，同时在内部实现一个匿名的skeletal implementation，将对对该接口的调用转发到匿名类中，起到“多继承”的效果
  +  simple implementation：提供一个非抽象的接口实现类，提供一个最简单、能work的实现，也允许被继承
  +  使用接口定义类型的缺点：不便于演进，一旦接口发布，如果想要增加功能（增加方法），则client将无法编译；而使用abstract class，则没有此问题，只需要提供默认实现即可
  +  小结
    +  通过接口定义类型，可以允许多实现（多继承）
    +  但是演进需求大于灵活性、功能性时，抽象类更合适
    +  提供接口时，提供一个skeletal implementation，同时审慎考虑接口设计
+  Item 19: 仅仅用interface去定义一个类型，该接口应该有实现类，使用者通过接口引用，去调用接口的方法
  +  避免用接口去定义常量，应该用noninstantiable utility class去定义常量
  +  相关常量的命名，通过公共前缀来实现分组
+  Item 20: Prefer class hierarchies to tagged classes
  +  tagged class: 在内部定义一个tag变量，由其控制功能的转换
  +  tag classes are verbose, error-prone, and inefficient
  +  而class hierarchy，不同功能由不同子类实现，公共部分抽象为一个基类，也能反映出各个子类之间的关系
+  Item 21: Use function objects to represent strategies
  +  只提供一个功能函数的类实例，没有成员变量，只需一个对象（单例），为其功能定义一个接口，则可以实现策略模式，把具体策略传入相应函数中，使用策略
  +  具体的策略实例通常使用匿名类定义，调用使用该策略的方法时才予以创建/预先创建好之后每次将其传入
+  Item 22: Favor static member classes over nonstatic
  +  有4种nested class：non-static member class; static member class(inner class); anonymous class; local class
  +  static member class
    +  经常作为helper class，和外部类一起使用
    +  如果nested class的生命周期独立于外部类存在，则必须定义为static member class，否则可能造成内存泄漏
    +  private static member class用处一：表示（封装）外部类的一些成员，例如Map的Entry内部类。
  +  non-static member class
    +  将持有外部类实例的强引用，可以直接引用外部类的成员和方法
    +  用处一：定义一个Adapter，使得外部内的实例，可以作为和外部类语义不同的实例来查看（访问），例如Collection的Iterator。
    +  如果nested class不需要引用外部类的成员和方法，则一定要将其定义为static，避免空间/时间开销，避免内存泄漏
  +  anonymous class
    +  当在非static代码块内定义时，会持有外部类的引用，否则不会持有
    +  限制
      +  只能在被声明的地方进行实例化
      +  无法进行instanceof测试
      +  不能用匿名类实现多个接口
      +  不能用匿名类继承一个类的同时实现接口
      +  匿名类中新添加的方法无法在匿名类外部访问
      +  不能有static成员
    +  应该尽量保持简短
    +  用处一：创建function object
    +  用处二：创建process object，例如：Runnable, Thread, TimberTask
    +  用处三：用于public static工厂方法，例如Collections类里面的一些工厂方法，很多是返回一个匿名的内部实现
  +  local class
    +  比较少用
    +  是否static取决于其定义的上下文
    +  可以在作用域内重复使用
    +  不能有static成员
    +  也应尽量保持简短
  +  小结
    +  四种nested class
    +  如果nested class在整个外部类内都需要可见，或者定义代码太长，应使用member class
    +  能static就一定要static，即便需要对外部类进行引用，对于生命周期独立于外部类的，也应该通过WeakReference进行引用，避免内存泄漏；至于生命周期和外部类一致的，则不必这样

## Generics
+  Item 23: Don’t use raw types in new code
  +  Java泛型，例如`List<E>`，真正使用的时候都是`List<String>`等，把E替换为实际的类型
  +  Java泛型从1.5引入，为了保持兼容性，实现的是伪泛型，类型参数信息在编译完成之后都会被擦除，其在运行时的类型都是raw type，类型参数保存的都是Object类型，`List<E>`的raw type就是`List`
  +  编译器在编译期通过类型参数，为读操作自动进行了类型强制转换，同时在写操作时自动进行了类型检查
  +  如果使用raw type，那编译器就不会在写操作时进行类型检查了，写入错误的类型也不会报编译错误，那么在后续读操作进行强制类型转换时，将会导致转换失败，抛出异常
  +  一旦错误发生，应该让它尽早被知道（抛出/捕获），编译期显然优于运行期
  +  `List`与`List<Object>`的区别
    +  前者不具备类型安全性，后者具备，例如以下代码
      ```java
        // Uses raw type (List) - fails at runtime!
        public static void main(String[] args) {
          List<String> strings = new ArrayList<String>();
          unsafeAdd(strings, new Integer(42));
          String s = strings.get(0); // Compiler-generated cast
        }

        private static void unsafeAdd(List list, Object o) {
          list.add(o);
        }
      ```
      不会报编译错误，但会给一个编译警告：`Test.java:10: warning: unchecked call to add(E) in raw type List list.add(o);`，而运行时则会发生错误。
    +  但如果使用`List<Object>`，即`unsageAdd`参数改为`List<Object> list, Object o`，则会报编译错误：`Test.java:5: unsafeAdd(List<Object>,Object) cannot be applied to (List<String>,Integer) unsafeAdd(strings, new Integer(42));`  
    +  因为`List<String>`是`List`的子类，但却不是`List<Object>`的子类。  
    +  并不是说这个场景应该使用`List<Object>`，这个场景应该使用`List<String>`，这里只是为了说明`List`和`List<Object>`是有区别的。
  +  `List` v.s. `List<?>`（unbounded wildcard types），当不确定类型参数，或者说类型参数不重要时，也不应该使用raw type，而应该使用`List<?>`
    +  任何参数化的List均是`List<?>`的子类，可以作为参数传入接受`List<?>`的函数，例如以下代码均是合法的：
      ```java
        void func(List<?> list) {
          ...
        }

        func(new List<Object>());
        func(new List<Integer>());
        func(new List<String>());
      ```
    +  持有`List<?>`的引用后，并不能向其中加入任何元素，读取出来的元素也是`Object`类型，而不会被自动强转为任何类型。
    +  如果`List<?>`的行为不能满足需求，可以考虑使用模板方法，或者`List<E extends XXX>`（bounded wildcard types）
  +  You must use raw types in class literals.
    +  `List.class`, `String[].class`, and `int.class` are all legal, but `List<String>.class` and `List<?>.class` are not.
  +  `instanceof`不支持泛型，以下用法是推荐的，但不应该将`o`强转为`List`
    ```java
      // Legitimate use of raw type - instanceof operator
      if (o instanceof Set) { // Raw type
        Set<?> m = (Set<?>) o; // Wildcard type
        ...
      }
    ```
  +  相关术语汇总  
  ![java_generic_terms.png](assets/java_generic_terms.png)
+  Item 24: Eliminate unchecked warnings
  +  当出现类型不安全的强制转换时（一般都是涉及泛型，raw type），编译器会给出警告，首先要做的是尽量消除不安全的转换，消除警告
  +  实在无法消除/确定不会导致运行时的`ClassCastException`，可以通过`@SuppressWarnings("unchecked")`消除警告，但不要直接忽略该警告
  +  使用`@SuppressWarnings("unchecked")`时，应该在注视内证明确实不存在运行时的`ClassCastException`；同时应该尽量减小其作用的范围，通常是应该为一个赋值语句添加注解
+  Item 25: Prefer lists to arrays
  +  arrays are covariant(协变): 如果`Sub`是`Super`的子类，那么`Sub[]`也是`Super[]`的子类
  +  generics are invariant(不变): 任意两个不同的类`Type1`和`Type2`，`List<Type1>`和`List<Type2>`之间没有任何继承关系
  +  考虑以下代码
  ```java
    // Fails at runtime!
    Object[] objectArray = new Long[1];
    objectArray[0] = "I don't fit in"; // Throws ArrayStoreException

    // Won't compile!
    List<Object> ol = new ArrayList<Long>(); // Incompatible types
    ol.add("I don't fit in");
  ```
  +  arrays are reified(具体化): array在运行时能知道且强制要求元素的类型
  +  generics are implemented by erasure(non-reifiable): 仅仅在编译时知道元素的类型
  +  数组和泛型同时使用时会受到很大限制
    +  以下语句均不能通过编译：`new List<E>[], new List<String>[], new E[]`；但是声明是可以的，例如`List<String>[] stringLists`
  +  non-reifiable type: 例如`E, List<E>, List<String>`，这些类型在运行时的信息比编译时的信息更少
  +  只有unbounded wildcard type才是reifiable的，如：`List<?>, Map<?, ?>`
  +  常规来说，不能返回泛型元素的数组，因为会报编译错误：`generic array creation errors`
  +  当泛型和`varargs`一起使用时，也会导致编译警告
  +  有时为了类型安全，不得不做些妥协，牺牲性能和简洁，使用List而不是数组
  +  把数组强转为non-reifiable类型是非常危险的，仅应在非常确定类型安全的情况下使用
+  Item 26: Favor generic types
  +  当需要一个类成员的数据类型具备一般性时，应该用泛型，这也正是泛型的设计场景之一，不应该用Object类
  +  但使用泛型有时也不得不进行cast，例如当泛型遇上数组
  +  总的来说把suppress数组类型强转的unchecked warning比suppress一个标量类型强转的unchecked warning风险更大，但有时出于代码简洁性考虑，也不得不做出妥协
  +  有时看似与item 25矛盾，实属无奈，Java原生没有List，ArrayList不得不基于数组实现，HashMap也是基于数组实现的
  +  泛型比使用者进行cast更加安全，而且由于Java泛型的擦除实现，也可以和未做泛型的老代码无缝兼容
+  Item 27: Favor generic methods
  +  泛型方法的类型参数在函数修饰符（可见性/static/final等）和返回值之间，例子：
  ```java
    // Generic method
    public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
        Set<E> result = new HashSet<>(s1);
        result.addAll(s2);
        return result;
    }
  ```
  +  recursive type bound
  ```java
    // Using a recursive type bound to express mutual comparability
    public static <T extends Comparable<T>> T max(List<T> list) {...}
  ```
  +  泛型方法要比方法使用者进行cast更加安全
+  Item 28: Use bounded wildcards to increase API flexibility
  +  考虑以下代码
  ```java
    public class Stack<E> {
        public Stack();
        public void push(E e);
        public E pop();
        public boolean isEmpty();

        public void pushAll(Iterable<E> src);
        public void popAll(Collection<E> dst);
    }

    Stack<Number> numberStack = new Stack<Number>();
    Iterable<Integer> integers = ... ;
    numberStack.pushAll(integers);

    Stack<Number> numberStack = new Stack<Number>();
    Collection<Object> objects = ... ;
    numberStack.popAll(objects);
  ```
  pushAll和popAll的调用均无法通过编译，因为尽管`Integer`是`Number`的子类，但`Iterable<Integer>`不是`Iterable<Number>`的子类，这是由泛型的invariant特性导致的，所以`Iterable<Integer>`不能传入接受`Iterable<Number>`参数的函数，popAll的使用同理
  +  bounded wildcards: `<? extends E>`, `<? super E>`, PECS stands for producer-extends, consumer-super. 如果传入的参数是要输入给该类型数据的，则应该使用extends，如果是要容纳该类型数据的输出，则应该使用super
  +  这很好理解，作为输入是要赋值给E类型的，当然应该是E的子类（这里的extends包括E类型本身）；而容纳输出是要把E赋值给传入参数的，当然应该是E的父类（同样包括E本身）
  +  返回值类型不要使用bounded wildcards，否则使用者也需要使用，这将会给使用者造成麻烦
  +  代码对于bounded wildcards的使用在使用者那边应该是透明的，即他们不会感知到bounded wildcards的存在，如果他们也需要考虑bounded wildcards的问题，则说明对bounded wildcards的使用有问题了
  +  有时候编译器的类型推导在遇到bounded wildcards会无法完成，这时就需要显示指定类型信息，例如：
  ```java
    public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2);

    Set<Integer> integers = ... ;
    Set<Double> doubles = ... ;
    //Set<Number> numbers = union(integers, doubles); //compile error
    Set<Number> numbers = Union.<Number>union(integers, doubles);  //compile pass
  ```
  +  Comparables are always consumers, so you should always use `Comparable<? super T>` in preference to `Comparable<T>`. The same is true of comparators, so you should always use `Comparator<? super T>` in preference to `Comparator<T>`.
  +  unbounded type parameter(`<E> ... List<E>`) v.s. unbounded wildcard(`List<?>`)：if a type parameter appears only once in a method declaration, replace it with a wildcard.
+  Item 29: Consider typesafe heterogeneous containers
  +  使用泛型时，类型参数是有限个的，例如`List<T>`，`Map<K, V>`，但有时可能需要一个容器，能放入任意类型的对象，但需要具备类型安全性，例如数据库的一行，它的每一列都可能是任意类型的数据
  +  由于`Class`类从1.5就被泛型化了，所以使得这种需求可以实现，例如：
  ```java
    // Typesafe heterogeneous container pattern - API
    public class Favorites {
        public <T> void putFavorite(Class<T> type, T instance);
        public <T> T getFavorite(Class<T> type);
    }
  ```
  +  通常这样使用的`Class`对象被称为type token，它传入函数，用来表述编译时和运行时的类型信息
  +  `Favorites`的实现也是很简单的：
  ```java
    // Typesafe heterogeneous container pattern - implementation
    public class Favorites {
        private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();

        public <T> void putFavorite(Class<T> type, T instance) {
            if (type == null)
            throw new NullPointerException("Type is null");
            favorites.put(type, instance);
        }

        public <T> T getFavorite(Class<T> type) {
            return type.cast(favorites.get(type));
        }
    }
  ```
  +  注意，这里的unbound wildcard并不是应用于Map的，而是应用于Class的类型参数，因此Map可以put key进去，而且key可以是任意类型参数的Class对象
  +  另外，Map的value类型是Object，一旦put到Map中去，其编译期类型信息就丢失了，将通过get方法的动态类型转换（cast）来重新获得其类型信息
  +  cast方法将检查类型信息，如果是该类型（或其子类），转换将成功，并返回引用，否则将抛出ClassCastException
  +  这一heterogeneous container实现有两个不足
    +  通过为put方法传入Class的raw type，使用者可以很轻易地破坏类型安全性，解决方案也很简单，在put时也进行一下cast：
    ```java
      // Achieving runtime type safety with a dynamic cast
      public <T> void putFavorite(Class<T> type, T instance) {
          favorites.put(type, type.cast(instance));
      }
    ```
    这样做的效果是使得想要破坏类型安全性的put使用者产生异常，而使用get的使用者则不会因为恶意put使用者产生异常。这种做法也被`java.util.Collections`包中的一些方法使用，例如命名为checkedSet, checkedList, checkedMap的类。
    +  这个容器内不能放入non-reifiable的类型，例如`List<String>`，因为`List<String>.class`是有语法错误的，`List<String>`, `List<Integer>`都只有同一个class对象：`List.class`；另外`String[].class`是合法的。
  +  `Favorites`使用的类型参数是unbounded的，可以put任意类型，也可以使用bounded type token，使用bounded时可能需要把`Class<?>`转换为`Class<? extends Annotation>`，直接用`class.cast`将会导致unchecked warning，可以通过`class.asSubclass`来进行转换，例子：
  ```java
    // Use of asSubclass to safely cast to a bounded type token
    static Annotation getAnnotation(AnnotatedElement element, String annotationTypeName) {
        Class<?> annotationType = null; // Unbounded type token
        try {
            annotationType = Class.forName(annotationTypeName);
        } catch (Exception ex) {
            throw new IllegalArgumentException(ex);
        }
        return element.getAnnotation(annotationType.asSubclass(Annotation.class));
    }
  ```

## Enums and Annotations
+  Item 30: Use enums instead of int constants
  +  类型安全
  +  可以为常量提供数据和方法的绑定
  +  可以遍历
  +  实现建议
    +  如果是通用的，应该定义为top level enum，否则应定义为内部类
    +  constant-specific method implementations
    ```java
      // Enum type with constant-specific method implementations
      public enum Operation {
          PLUS   { double apply(double x, double y){return x + y;} },
          MINUS  { double apply(double x, double y){return x - y;} },
          TIMES  { double apply(double x, double y){return x * y;} },
          DIVIDE { double apply(double x, double y){return x / y;} };
          abstract double apply(double x, double y);
      }
    ```
    +  结合constant-specific data
    ```java
      // Enum type with constant-specific class bodies and data
      public enum Operation {
          PLUS("+") {
              double apply(double x, double y) { return x + y; }
          },
          MINUS("-") {
              double apply(double x, double y) { return x - y; }
          },
          TIMES("*") {
              double apply(double x, double y) { return x * y; }
          },
          DIVIDE("/") {
              double apply(double x, double y) { return x / y; }
          };

          private final String symbol;
          Operation(String symbol) { this.symbol = symbol; }

          @Override public String toString() { return symbol; }
          abstract double apply(double x, double y);
      }
    ```
    +  If switch statements on enums are not a good choice for implementing con- stant-specific behavior on enums, what are they good for? Switches on enums are good for augmenting external enum types with constant-specific behavior.
  +  A minor performance disadvantage of enums over int constants is that there is a space and time cost to load and initialize enum types.
  +  所以，在安卓设备（手机、平板）上，应该避免使用enum，减小空间和时间的开销
+  Item 31: Use instance fields instead of ordinals
  +  每个enum的常量都有一个`ordinal()`方法获取其在该enum类型中的位置，但该方法只应该在实现`EnumSet`, `EnumMap`等类型的时候被使用，其他情形都不应该被使用
  +  如果需要为每一个常量绑定一个数据，可以使用instance field实现，如果需要绑定方法，则可以用constant-specific method implementations，参考上一个item
+  Item 32: Use EnumSet instead of bit fields
  +  bit fields的方式不优雅、容易出错、没有类型安全性
  +  EnumSet则没有这些缺点，而且对于大多数enum类型来说，其性能都和bit field相当
  +  通用建议：声明变量时，不要用实现类型，应该用接口类型，例如，应该用`List<Integer>`而不是`ArrayList<Integer>`
  +  EnumSet并非immutable的，可以通过`Conllections.unmodifiableSet`来封装为immutable，但是代码简洁性与性能都将受到影响
+  Item 33: Use EnumMap instead of ordinal indexing
  +  同前文所述，应该避免使用ordinal。当需要用enum作为下标从数组获取数据时，可以换个角度思考，以enum作为key从map里面获取数据。
  +  数组和泛型不兼容，因此使用数组也会导致编译警告；而且ordinal的值本来就不是表达index含义的，极易导致隐蔽错误
  +  EnumMap内部使用数组实现，因此性能和数组相当
  +  使用数组也会导致程序可扩展性下降，考虑以下两种实现
  ```java
    // Using ordinal() to index array of arrays - DON'T DO THIS!
    public enum Phase {
        SOLID, LIQUID, GAS;

        public enum Transition {
          MELT, FREEZE, BOIL, CONDENSE, SUBLIME, DEPOSIT;

          // Rows indexed by src-ordinal, cols by dst-ordinal
          private static final Transition[][] TRANSITIONS = {
                  { null,    MELT,     SUBLIME },
                  { FREEZE,  null,     BOIL    },
                  { DEPOSIT, CONDENSE, null    }
          };

          // Returns the phase transition from one phase to another
          public static Transition from(Phase src, Phase dst) {
            return TRANSITIONS[src.ordinal()][dst.ordinal()];
          }
        }
    }

    // Using a nested EnumMap to associate data with enum pairs
    public enum Phase {
        SOLID, LIQUID, GAS;

        public enum Transition {
            MELT(SOLID, LIQUID), FREEZE(LIQUID, SOLID),
            BOIL(LIQUID, GAS),   CONDENSE(GAS, LIQUID),
            SUBLIME(SOLID, GAS), DEPOSIT(GAS, SOLID);

            final Phase src;
            final Phase dst;

            Transition(Phase src, Phase dst) {
                this.src = src;
                this.dst = dst;
            }

            // Initialize the phase transition map
            private static final Map<Phase, Map<Phase,Transition>> m =
                new EnumMap<Phase, Map<Phase,Transition>>(Phase.class);
            static {
                for (Phase p : Phase.values())
                    m.put(p,new EnumMap<Phase,Transition>(Phase.class));
                for (Transition trans : Transition.values())
                    m.get(trans.src).put(trans.dst, trans);
            }

            public static Transition from(Phase src, Phase dst) {
                return m.get(src).get(dst);
            }
        }
    }
  ```
  当需要增加`Phase`时，前者需要谨慎地修改`TRANSITIONS`数组的内容（这一步骤容易出错），而后者则只需要增加相应`Transition`即可，`from`函数的逻辑完全不受影响。
+  Item 34: Emulate extensible enums with interfaces
  +  当enum遇到可扩展性时，总是一个糟糕的问题；扩展类是基础类的实例，但反过来不是，这一点很让人困惑；想要枚举所有基础类和扩展类的enum对象时，并没有一个很好地办法；
  +  而对于可扩展性的需求，是真实存在的，例如：operation codes (opcodes)
  +  实现方式是通过定义一个接口，enum类型（基础与扩展）均实现该接口，而在使用enum的地方，接收这个接口作为参数
  +  enum类型是不可扩展的，但是interface具备可扩展性，如果API使用接口而非实现去代表operation，API就有了可扩展性
  +  泛型高级用法：`<T extends Enum<T> & Operation> ... Class<T>`，T类型是enum类型，且是`Operation`子类
  +  这一方式的不足：enum类型对接口的实现是不能继承的
+  Item 35: Prefer annotations to naming patterns
  +  在1.5之前，naming patterns很常见，在JUnit中都是这样，例如要求测例方法一`test`开头
  +  naming patterns有很多问题
    +  拼写错误不能及时发现
    +  无法保证naming patterns只在正确的场景使用，例如可能有人以`test`开头命名测例类，方法却没有，JUnit则不会运行测例
    +  没有值/类型信息，编译器无法提前发现问题
  +  使用annotations可以很好的解决这些问题，但是annotations的功能也是有限的
    +  `@Retention(RetentionPolicy.RUNTIME)`能限定其保留时期
    +  `@Target(ElementType.METHOD)`能限定其应用的程序元素
    +  还有其他meta-annotations，如`@IntDef`
  +  annotations接收的参数如果是数组，为其赋值一个单独的元素也是合法的
+  Item 36: Consistently use the Override annotation
  +  `@Override`会使得重写的准确性得到检查
  +  重载和重写的区别：一个只是函数名一样，通过参数列表决定执行哪个版本，是编译时多态；一个是通过虚函数机制实现，是运行时多态；
+  Item 37: Use marker interfaces to define types
  +  定义一个空的接口，表明某个类型的属性，例如`Serializable`
  +  另一种方式是使用annotation，表明者其具有某种属性
  +  marker interface的优点
    +  定义了一个类型，可以进行instanceof判断，可以声明参数类型
    +  比annotation更简洁
  +  marker annotation的优点
    +  当一个类型（通过interface或者annotation）被声明后，如果想要加入更多的信息，annotation更方便，即annotation对修改是开放的，因为它的属性可以有默认值，而interface则不行，定义了方法就必须实现
    +  annotation可以被应用到更多代码的元素中，不仅仅是类型
  +  实现建议
    +  如果仅仅只应用于类型，则应该优先考虑annotation
    +  如果希望mark的对象被限定于某个接口的实例（即为一个接口增加另外一种语义，却不改变其API），可以考虑使用marker interface

## 函数
+  Item 38: Check parameters for validity
  +  一个函数（包括构造函数）首先要做的事情就是验证参数合法性，如果不合法则应该抛出相应异常，这是对“尽早发现错误尽早抛出”原则的遵循，否则等到错误发生时将可能难以判断错误的根源所在，甚至程序不会显式报错，而是执行了错误的行为，导致更严重的后果
  +  不由被调用函数使用，而是存起来留作后用的参数，更加要检查其合法性
  +  Javadoc里面应该注明`@throw`项，并说明原因
  +  非公开的API（private或package private），则不应该通过抛异常来报错，应该采用`assert`，assert可以通过配置虚拟机参数开启或关闭，如果关闭则不会被执行
  +  灵活运用，设计API时，就应该尽量设计得通用一些，即可以接受更大范围的参数，毕竟检查参数也是有开销的
  +  另外可以考虑抛出`RuntimeException`的子类，因为这样的异常不用放到函数的异常表中，函数的使用者也不用必须`try-catch`或者`throw`，但doc一定要写明
+  Item 39: Make defensive copies when needed
  +  编码一大原则：永远不要信任用户（调用方）输入的数据，也不要信任它们不会篡改返回的数据，因此defensive copy很有必要
  +  编写一个类时，如果成员变量是mutable的，那么就需要在构造函数（或者setter）中进行深拷贝，并且，先拷贝，再验证已拷贝数据的合法性（既不是先验证，也不是验证传入的数据，避免TOCTOU attack）
  +  另外深拷贝时，传入对象的类如果不是final的，就不能用clone方法进行拷贝，因为不能保证clone方法返回的就正好是这个类的实例（有可能会是恶意的子类）
  +  为mutable成员提供getter方法时，返回前也要进行深拷贝，但此时可以用clone方法，因为我们确定成员就是我们想要的类的对象
  +  java内建的Map, Set等容器，实现上是没有进行深拷贝的，因为是泛型，所以put进去或者get出来的时候，编译期都不知道具体是什么类型，是无法调用构造函数的，如果想要测试这一问题，需要确定key和value的类型都是mutable的，如果测`Map<String, Integer>`，那结果肯定是错误的，但如果测`Map<StringBuilder, Date>`，就可以知道确实如此；所以如果要把用户传入的数据放入Map，且key/value是mutable的，那么就需要在put之前进行深拷贝，否则可能会被用户attack
  +  长度非零的数组都是mutable的
  +  尽量使用immutable的成员就可以省去深拷贝带来的性能开销
  +  如果确实信任用户，就可以把深拷贝省去，但一定要在文档内说明，例如：wrapper模式，如果用户恶意，那损害的也就仅仅是其自身；或者用户都是自己的代码，可以确信安全。
+  Item 40: Design method signatures carefully
  +  命名要合理，可理解：清除表达函数的功能；符合常识；保持风格一致；
  +  类/接口的成员方法数量不要太多，否则会令人难以理解，而且不利于测试、维护
  +  不要随便提供helper方法，只有当很有必要时才提供
  +  避免过长参数列表（不多于4个），尤其是参数类型相同，否则既难记（倒还好），又可能引起隐晦的bug（传入参数顺序错了，编译不报错，运行时行为确是错的）
    +  可以通过把参数列表过长的方法拆分为几个方法，但要避免导致方法过多
    +  创建helper类，容纳作用相关联的的参数
    +  类似于构造对象的Builder模式，为函数的调用创建一个builder
  +  参数类型，使用interface，而不是实现类
  +  对于起控制作用的参数，使用二值enum，而不是boolean，便于扩展；对于安卓来说，可以通过`@IntDef`辅助定义int常量，模拟enum
+  Item 41: Use overloading judiciously
  +  慎用重载，重载（overload）与重写（override）的区别可以见上文，简言之，前者编译时多态，后者运行时多态
  +  重载是编译时多态，版本选择在编译期完成，根据编译期参数的类型信息来进行决策
  +  建议不要用参数类型来设计不同的重载版本，应该通过参数列表长度，或者没有父子类关系的不同参数类型，例如接受int和float的类型，后者也还是可能会有问题
+  Item 42: Use varargs judiciously
  +  varargs的原理是调用时首先创建一个数组，然后把传入的参数放入数组，数组长度为参数个数
  +  一个方法需要0或多个同类型数据这个需求很常见，然而也有另一个很常见的需求：需要一个或多个同类型数据，此时单纯用varargs不太优雅，可以让方法先接受一个数据，在接受一个varargs
  +  varargs最初是为了printf和反射设计的
  +  可以通过把传入参数从一个数组改为varargs，来改良该方法（变得更灵活），而且对已有代码“无影响”，`Arrays.asList`便是一个例子，但接受varargs最初是为了打印数组内容设计的，而不是为了把多个数据变成一个List
  +  Don’t retrofit every method that has a final array parameter; use varargs only when a call really operates on a variable-length sequence of values.
  +  以下两种函数声明都可能会产生问题：
  
    ```java
    ReturnType1 suspect1(Object... args) { }
    <T> ReturnType2 suspect2(T... args) { }
    ```
    
     如果传入一个基本类型的数组进去（例如int[]），那么这两个方法接受的都是一个int[][]，即相当于接受了一个只有一个元素的数组，而这个数组的数据类型是int[]！而如果传入一个对象的数组，则相当于传入了数组长度个数的varargs。`Arrays.asList`方法就存在这个问题！
  +  varargs也存在性能影响，因为每次调用都会创建、初始化一个数组。如果为了不失API灵活性，同时大部分调用的参数个数都是有限个，例如0~3个，那么可以声明5个重载版本，分别接受0~3个参数，另外加一个3个参数+varargs的版本
+  Item 43: Return empty arrays or collections, not nulls
  +  可能有人认为返回null能减小内存开销，然：
    +  永远不要过度考虑性能问题，只有当profiling显示瓶颈就是这里的时候，再考虑性能优化与代码优雅性的牺牲，当然，无副作用的优化肯定尽早采纳
    +  可以每次需要返回空数组/集合时，返回同一个空数组/集合，这样就只需要一次内存分配
  +  Collection的`<T> T[] toArray(T[] a)`方法，可以每次调用时传入一个空数组，因为该方法保证如果集合元素可以被放入提供的参数数组中，将不会分配新内存，当放不下时才会分配
  +  下面实现返回集合的值的方式也是值得借鉴的
  
  ```java
    // The right way to return a copy of a collection
    public List<Cheese> getCheeseList() {
      if (cheesesInStock.isEmpty())
        return Collections.emptyList(); // Always returns same list
      else
        return new ArrayList<Cheese>(cheesesInStock);
    }
  ```
  +  Item 44: Write doc comments for all exposed API elements
    +  对于API暴露的部分（类、接口、方法、成员等），都应该先写好文档；为了提高代码的可维护性，未暴露的部分也应该写好文档；
    +  每个方法的文档的内容，应该是描述该方法与调用者之间的约定，不必是实现细节，细节可以看代码，约定则是使用者关心的东西；设计为被继承的类，方法文档应该描述该方法做了什么，而不是怎么做的；
    +  方法的文档中，应该描述约定的前提条件，执行后产生的影响，尤其是对于“系统”（或者说这个对象）状态的影响；不符合前提条件的情形将抛出异常；
    +  更多细节
      +  `@param`, `@return`, `@throws` 描述不要句号结尾
      +  `@throws` 的描述应该以if开头，其他都应该是名词描述
      +  `@{code}`与`@{literal}`
      +  有泛型时，需要说明每个类型参数
      +  enum类型要为每个常量注释含义
      +  annotation的定义，要为每个成员/参数注释含义
      +  线程安全性说明，可见性说明，序列化说明
      
## 编程通用
+  Item 45: Minimize the scope of local variables
  +  在变量第一次使用的时候进行声明，声明时尽量就进行初始化
  +  因此也更倾向于使用for-loop，而不是while-loop，因为后者需要使用while-loop外定义的控制变量
  +  for-loop的终结条件变量n，也应该在循环变量i初始化时计算，避免重复计算
  +  保持方法简短，一个方法只做一件事
+  Item 46: Prefer for-each loops to traditional for loops
  +  优点之一：可以避免一些容易犯的bug
    ```java
      // Can you spot the bug?
      enum Suit { CLUB, DIAMOND, HEART, SPADE }
      enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
                  NINE, TEN, JACK, QUEEN, KING }
      ...
      Collection<Suit> suits = Arrays.asList(Suit.values());
      Collection<Rank> ranks = Arrays.asList(Rank.values());
      List<Card> deck = new ArrayList<Card>();
      for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
          for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
              deck.add(new Card(i.next(), j.next()));
    ```
    
    `i.next()`在内层循环被调用了多次。以下写法则直观且不易出错：
    
    ```java
      // Preferred idiom for nested iteration on collections and arrays
      for (Suit suit : suits)
          for (Rank rank : ranks)
              deck.add(new Card(suit, rank));
    ```
  +  此种写法不仅可用于集合和数组，任何实现`Iterable`接口的类都可以用于冒号后面的部分
  +  缺点
    +  有性能代价！一定会创建Iterator，对于安卓开发，不建议如此。
    +  不能在for-each语法中进行remove，用Iterator遍历时，能remove
    +  遍历过程中替换原有元素
    +  Parallel iteration
+  Item 47: Know and use the libraries
  +  don't reinvent the wheel
  +  视野！
+  Item 48: Avoid float and double if exact answers are required
  +  float和double设计为用于科学计算，“精确近似”，需要确切结果的，不要使用，例如：货币相关！应该使用BigDecimal, int, 或者long。
  +  BigDecimal使用有些不方便，性能也比primitive类型低
+  Item 49: Prefer primitive types to boxed primitives
  +  两种类型的区别
    +  boxed类型，除了包含数值外，还有不同的唯一标示，即值一样，对象可以不一样，这一点很重要！
    +  boxed类型，比primitive类型多一个值，null
    +  boxed类型，时间、空间效率均低一些
  +  caveats
    +  有些操作会auto-unbox，例如：加减乘除，大小比较，但判等（`==`）不会！
    +  Applying the `==` operator to boxed primitives is almost always wrong.
    +  boxed类型，值为null时，会unbox为什么呢？会抛出`NullPointerException`
    +  当boxed和primitive出现在同一个运算中，boxed类型会auto-unbox（包括判等）
    +  大量重复的box/unbox会导致性能大幅下降
  +  使用场景与注意事项
    +  放到标准集合里面，必须是boxed类型
    +  作为类型参数（泛型），必须是boxed类型
    +  auto-box是安全的，也能省去繁琐的代码，但是auto-unbox则可能引起隐蔽的错误
+  Item 50: Avoid strings where other types are more appropriate
  +  Strings are poor substitutes for other value types. 只有当数据确实就是文本时，才适合用String。
  +  Strings are poor substitutes for enum types.
  +  Strings are poor substitutes for aggregate types. 把一系列数据转化为一个String（序列化），然后再反序列化，也应该用Json，如果自定义分隔符，既不优雅，也不安全。
  +  Strings are poor substitutes for capabilities. capability是一种称呼，通常就是说不同的对象，凭借一个key去同一个地方保存、获取数据；如果用String，那么如果内容相同，那key就会冲突，不安全；ThreadLocal的发展史*。
+  Item 51: Beware the performance of string concatenation
  +  用`+`连接n个String，时间复杂度为`O(n^2)`，因为String是immutable的，所以每次拼接都会拷贝两者的内容
  +  使用`StringBuilder`进行拼接操作；不过对于安卓开发来说，基本没什么影响，因为在打包的过程中，这一优化会自动完成；
+  Item 52: Refer to objects by their interfaces
  +  如果有接口，那么函数参数、返回值、成员变量、局部变量，都应该使用接口来保持对象的引用，只有在通过构造函数创建对象时才应该引用具体的实现类型；面向接口编程更广义的实践；
  +  面向接口编程使得程序更加灵活，切换实现类非常简单；但如果代码功能/正确性依赖于实现类特有的特性，那么切换时就需要仔细考虑一下；
  +  当然，如果对应功能的接口不存在，那直接引用该类当然是可以的；value type; class-based framework; 或者实现类提供了接口不存在的功能
+  Item 53: Prefer interfaces to reflection
  +  反射可以访问私有成员
  +  反射可以调用编译时不存在的类的方法，当然需要运行时已经加载
  +  但是反射也是有代价的
    +  编译期的类型检查完全失效，类型安全性丧失
    +  反射代码繁琐且易出错，当然这一点有一些好的框架可以避免，例如[JOOR](https://github.com/jOOQ/jOOR)
    +  性能下降，反射调用性能会低很多
  +  反射常用的场景
    +  class browsers, object inspectors, code analysis tools, and interpretive embedded systems, remote procedure call (RPC) systems
    +  反射功能强大，也有一些不足，如果合适利用，还是非常方便的
    +  例如编译期有些类尚未获得，但是如果有其父类/接口，则可以声明为父类/接口，只通过反射创建实例，其余代码都无需反射
+  Item 54: Use native methods judiciously
  +  设计之初的三大用途
    +  访问平台相关的功能，例如registries and file locks
    +  访问老的C/C++版本的库，访问老的数据
    +  追求性能
  +  近年来JVM/Java的发展，性能已有很大改善，追求性能而使用JNI通常来说都已经没必要了
  +  JNI的劣势
    +  不安全，内存管理不受JVM控制了，溢出等问题都有可能发生了
    +  平台相关
    +  难以调试
    +  Java和native层的交互是有开销的
    +  native代码比Java代码更难懂
  +  对于安卓应用开发来说，JNI还有一点就是隐藏实现，Java代码反编译非常容易，而native代码则难一些
+  Item 55: Optimize judiciously
  +  只有当确实需要时，才考虑性能优化，当然一些常见的范式，初次编码时就应该遵循
  +  Strive to write good programs rather than fast ones; speed will follow.
  +  Strive to avoid design decisions that limit performance.
  +  Consider the performance consequences of your API design decisions.
  +  It is a very bad idea to warp an API to achieve good performance. 
  +  当确实需要优化性能时：measure performance before and after each attempted optimization.
  +  找到原因后，首先考虑的是算法的优化，然后是上层的优化
  +  在进行优化前，对程序进行profiling，确定瓶颈，否则可能浪费精力反而性能下降
+  Item 56: Adhere to generally accepted naming conventions
  +  包名要体现出组件的层次结构，全小写
  +  公布到外部的，包名以公司/组织的域名开头，例如：edu.cmu, com.sun
  +  ...
  
## 异常处理
+  Item 57: Use exceptions only for exceptional conditions
  +  exceptions are, as their name implies, to be used only for exceptional conditions; they should never be used for ordinary control flow.
  +  A well-designed API must not force its clients to use exceptions for ordinary control flow.
    +  如果一个类的某个方法，依赖于该类当前处于某个特定状态，则应该提供一个单独的状态检查方法，例如Iterator的next和hasNext方法
    +  另外如果不提供状态检查方法，也可以让方法在异常状态下，返回一个特定的非法值
    +  如果该类被并发访问，且访问时未进行互斥处理，则必须使用返回非法值的方式；另外考虑到性能因素，也更倾向于返回非法值；其他情况下，都应该使用状态检查方法，可读性更好，更容易检查错误；
+  Item 58: Use checked exceptions for recoverable conditions and runtime exceptions for programming errors
  +  use checked exceptions for conditions from which the caller can reasonably be expected to recover.
  +  unchecked exception: `RuntimeException`, `Error`通常都不需要、也不应该catch
  +  Use runtime exceptions to indicate programming errors. 通常用于表示程序运行的状态违背了前提条件，违背了API的约定
  +  all of the unchecked throwables you implement should subclass RuntimeException
+  Item 59: Avoid unnecessary use of checked exceptions
  +  如果即便合理的调用了API也会遇到异常情形，并且捕获异常之后能够进行一些有意义的操作，才应该使用checked exception，其他情况下都应该使用RuntimeException
  +  通常，如果一个方法会抛出checked exception，都可以将其拆分为两个方法，一个用于判断是否会抛出异常，另一部分用于处理正常情况，如果不符合约定，就抛出RuntimeException，这样使得API更易用，也更灵活；但是要考虑状态检查和执行之间，是否可能从外部其他线程修改对象的状态；
+  Item 60: Favor the use of standard exceptions
  +  IllegalArgumentException, IllegalStateException, NullPointerException, IndexOutOfBoundsException, ConcurrentModificationException, UnsupportedOperationException
+  Item 61: Throw exceptions appropriate to the abstraction
  +  exception translation: higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction.

    ```java
    // Exception Translation
    try {
        // Use lower-level abstraction to do our bidding
        ...
    } catch(LowerLevelException e) {
        throw new HigherLevelException(...);
    }
    ```
  +  While exception translation is superior to mindless propagation of excep- tions from lower layers, it should not be overused.
+  Item 62: Document all exceptions thrown by each method
  +  Always declare checked exceptions individually, and document precisely the conditions under which each one is thrown using the Javadoc @throws tag. 不要通过声明抛出多个异常的父类来实现抛出多种异常的效果。
  +  要为每个方法可能抛出的unchecked exception写文档，但是不要将这些异常放到方法声明的异常表中去。便于API使用者区分checked和unchecked exception。
  +  如果一个类的很多方法都抛出同一个异常，那么可以将文档放到class doc中，而不是method doc中。
+  Item 63: Include failure-capture information in detail messages
  +  To capture the failure, the detail message of an exception should contain the values of all parameters and fields that “contributed to the exception.”
  +  良好设计的Exception类，应该把它需要的详细信息都作为构造函数的参数，而不是统一接收String参数；这样将把生成有意义的detail信息的任务集中在了Exception类本身，而不是其使用者。
  +  checked exception可以为failure-capture information提供访问方法，以便于使用者在程序上进行恢复处理；虽然unchecked exception通常不会在程序中进行恢复，但是提供同样的方法也是建议的做法。
+  Item 64: Strive for failure atomicity
  +  Generally speaking, a failed method invocation should leave the object in the state that it was in prior to the invocation. 满足此属性的方法称为 failure atomic。
    +  immutable对象是最简单的实现方法
    +  mutable对象要达到此效果，就需要在进行操作前，对所有的参数、field进行检查
    +  有可能无法在函数的第一部分进行检查，但是一定要在对对象进行修改之前进行检查
    +  还有一种不太常见的方式：函数内部捕获异常，异常发生之后先回退对象的状态，再把异常抛出去
    +  还可以先创建一个临时的对象，在临时对象上进行操作，成功后替换原对象的值
  +  有的情况下，failure atomic是不可能的，所以也就没必要为此做出努力了
  +  有的情况下，为了failure atomic，会增加很多额外的开销、复杂度，也就不太必要了
  +  当方法不满足failure atomic时，需要在文档中进行说明
+  Item 65: Don’t ignore exceptions
  +  An empty catch block defeats the purpose of exceptions
  +  At the very least, the catch block should contain a comment explaining why it is appropriate to ignore the exception.
  +  忽略异常，可能导致程序在其他不相关的地方失败/崩溃，这时将很难找到/解决根本问题
  
## 并发
+  Item 66: Synchronize access to shared mutable data
  +  `synchronized`不仅是为了保证每个线程访问/执行时，看到的都是“正常状态”的对象（所谓正常就是没有发生多线程同时未加同步的写同一个对象，导致其状态不一致）；还能保证每个线程看到的都是最新的对象；
  +  Java语言保证了基本类型中除了long和double的访问都是原子性的，并发写这些类型的数据而不进行同步控制，也不会有问题
  +  有人建议访问具有原子性操作属性的对象无需进行同步控制，还能提升性能，纯属一派胡言
  +  Java语言不会保证并发访问时，其他线程写的值能立即被读的线程感知，所以同步操作不仅仅是为了互斥访问，也是为了保证多线程之间看到的始终是最新的值
  +  上述问题的根本原因就是[Java memory model](Android-Java/JSR133.md)
  +  一个简单、常见、易错的例子
    +  如何停止后台线程？首先不能调用`Thread.stop`方法，这个方法会导致data corruption
    +  常用的方法就是用一个`boolean`变量，后台线程根据其值决定是否停止，而主线程想要停止后台线程时，修改这个变量的值即可
    +  `boolean`的读写操作是原子性的，并发访问不加同步，不会导致data corruption，但是却无法保证主线程对变量的修改能及时被后台线程感知，甚至无法保证能被感知
    +  指令重排，如果`done`就是个普通声明的`boolean`，以下变换在Java memory model下是允许的
        ```java
        while (!done)
          i++;
        
        //==>
        if (!done)
          while (true)
            i++;
        ```
    +  可想而知，如果未进行同步操作，后台线程将永远不会停止
    +  解决方法有两种
      +  为`done`的读写访问都加上`synchronized`，注意，读写都需要，否则没有数据同步（communication）的效果；由于`boolean`的读写访问是原子性的，所以这里的`synchronized`仅仅起数据同步的作用；
      +  声明`done`的时候加上`volatile`关键字，`volatile`没有互斥的作用，仅仅是起数据同步的作用，在这里正好满足需求；这种方式性能比上一种要好一些；
  +  使用`volatile`需要格外谨慎，因为它并没有互斥作用，如果声明一个`volatile int`，然后对其进行`++`操作，那将会导致data corruption，因为`++`不是原子性的
  +  对于这种需求，可以声明为`synchronized int`；更好的方式是使用`java.util.concurrent.atomic`包下的类，安全，高效；
  +  更根本的解决方式就是不要多线程共享mutable对象，而是共享immutable对象；甚至不要多线程共享数据；
  +  引入框架/库时，需要考虑一下它们是否会引入多线程问题
  +  effectively immutable：对象不是真的immutable，但是对象分享出去之后，就不会再改变了；当然这个还是很危险的，因为并没有强制的机制保证不会被修改；
  +  小结：多线程访问共享变量时，读和写都需要进行同步操作
+  Item 67: Avoid excessive synchronization
  +  在同步代码块中，不要调用可能被重写的方法，更不要调用使用者传入对象的方法，因为这些代码是不可控的，可能导致异常、死锁、data corruption
  +  对于Observer模式中的observer list，Java 1.5之后有一个单独优化的高效并发容器：`CopyOnWriteArrayList`，每次写（添加、删除）操作都会从内部的数组创建一份新的拷贝，读（遍历）操作时完全不用加锁，对于读多写少的场景性能很好
  +  一个总的原则是，在同步代码块中，执行尽可能少的操作；如果有耗时操作，应该在保证安全的前提下，尝试各种手段，将其移出同步块；
  +  过度同步的性能影响
    +  丧失了多核CPU的并行性，获得锁的开销倒是其次
    +  任何时刻都需要保证每个CPU核心之间的数据同步，这有不小的开销
    +  限制了JVM的代码优化空间
  +  共享数据的并发访问，一定要保证线程安全；如果可以在类内部，通过少量/高效的同步块保证，就不要把整个类的任何操作都加锁；如果做不到，那就不要进行任何同步，把这个责任交给使用者，给他们优化的空间，但一定要在文档中说明；
  +  如果`static`成员可以被某些方法修改，那一定要为它们加锁，因为这种情况下使用者无法保证线程安全性
+  Item 68: Prefer executors and tasks to threads
  +  Executor Framework
  
  ```java
  ExecutorService executor = Executors.newSingleThreadExecutor();
  executor.execute(runnable);
  executor.shutdown();
  ```
  +  `Executors`提供了多个工厂方法，创建`ExecutorService`，还可以直接使用`ThreadPoolExecutor`，对线程池做更精细的控制
  +  如果程序负载轻，可以使用`Executors.newCachedThreadPool`，任务提交时如果没有空闲线程，将创建新的线程；如果负载重，用`Executors.newFixedThreadPool`更合适；
  +  不仅不应该自己实现任务队列，甚至都应该避免直接使用线程，而是使用Executor Framework；
  +  任务和机制被分别抽象了，前者为`Runnable`和`Callable`，后者则是executor service；
  +  `java.util.Timer`也尽量不要用了，可以使用`ScheduledThreadPoolExecutor`；
+  Item 69: Prefer concurrency utilities to wait and notify
  +  正确使用`wait`和`notify`有难度，而Java又提供了更高层的抽象，何乐而不用呢？
  +  `java.util.concurrent`包主要包含三块：
    +  Executor Framework
    +  concurrent collections
    +  synchronizers
  +  concurrent collections提供了标准容器的多线程高性能版本，它们内部进行了同步互斥操作，保证正确性；外部使用的时候，无需加锁，否则只会导致性能下降；
    +  concurrent collections中的每一种实现，可能都有性能优化的侧重点，可能有的是多读少写高效，例如`CopyOnWriteArrayList`，所以使用时需要了解清楚其试用场景；
    +  除非有明确的理由，否则，优先使用`ConcurrentHashMap`，而不是`Collections.synchronizedMap`或者`Hashtable`；也尽量避免在使用者那端进行同步操作；
    +  有的concurrent collections提供了block操作接口，例如`BlockingQueue`，从中取数据的时候，如果队列为空，线程将等待，新的数据加入后，将自动唤醒等待的线程；大部分的`ExecutorService`都是采用这种方式实现的；
  +  Synchronizers: `CountDownLatch`, `Semaphore`, `CyclicBarrier`, `Exchanger`
    +  `CountDownLatch`: 多个线程等待另外一个或多个线程完成某种工作
    +  注意thread starvation deadlock问题
    +  `Thread.currentThread().interrupt()` idiom：异常可能从其他线程抛出？用此方法回到原来的线程？
    +  计时的话，用`System.nanoTime()`而不是`System.currentTimeMillis()`，前者更准确，更明确
  +  如果非要用`wait`和`notify`，注意以下几点：
    +  Always use the wait loop idiom to invoke the wait method; never invoke it outside of a loop.
    +  wait前的条件检查可以保证不会死锁，wait后的检查可以保证安全
    +  通常情况下都应该使用`notifyAll`
+  Item 70: Document thread safety
  +  一个方法的声明中加了`synchronized`并不能保证它是线程安全的，并且Javadoc也不会把这个关键字输出到文档中
  +  线程安全也分好几个层次，文档中应该说明类/方法做到了何种程度上的线程安全
  +  线程安全的分类
    +  immutable，对象创建后不可修改，无需进行外部的同步操作（互斥访问控制或许更恰当）；例如：`String`, `Long`, `BigInteger`；
    +  unconditionally thread-safe，对象可变，但是其内部进行了正确的同步操作，无需外部进行同步；例如：`ConcurrentHashMap`；
    +  conditionally thread-safe，和绝对线程安全类似，但是有些方法需要进行外部的同步操作；例如：`Collections.synchronized`返回的容器，它们的iterator使用时需要进行同步；
    +  not thread-safe，类自身没有任何同步操作，需要使用者自己保证线程安全；例如：`ArrayList`；
    +  thread-hostile，由于类的实现原因，使用者无论如何也无法保证线程安全，例如未加同步的修改static成员；例如：`System.runFinalizersOnExit`；
  +  jsr-305引入了几个注解：`Immutable`, `ThreadSafe`, `NotThreadSafe`，对应上述前四种情形，绝对线程安全与条件线程安全都属`ThreadSafe`，对于条件线程安全还应在文档中说明何种情况下是需要外部进行同步的；
  +  如果一个类，将它用于`synchronized`的对象暴露出去了，那是很危险的，通常的做法是，内部创建一个`Object`实例，将其用于`synchronized`，但这种方式通常只适用于unconditionally thread-safe的实现。
  
    ```java
      // Private lock object idiom - thwarts denial-of-service attack
      private final Object lock = new Object();
      public void foo() {
        synchronized(lock) {
          ...
        }
      }
    ```
