#依赖注入（以Dagger 2为例）

##Custom scope
Scope提供了作用域范围内的单例特性，对象的引用在scope内会一直存在，例如：ApplicationScope，ActivityScope。  
Scope其实就是一个Component的对象，Scope的注解就是在Component上的。