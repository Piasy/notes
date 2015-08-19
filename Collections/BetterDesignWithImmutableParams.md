#函数调用时，传递参数应该是不可变的（Immutable）
考虑以下代码：
```java
Hashmap map = new HashMap(); 
map.put("key", "value") 
service.doSomething(map); 
map.clear() 
```
测试时想要验证doSomething调用时的参数内容（状态），使用Mockito的ArgumentCaptor，capture到的都将是空的map，因为capture到的对象，在调用doSomething后，又被修改了（clear）。  
一方面目前Mockito的实现，并非capture时立即创建参数的副本，而是直接持有其引用，所以后面的修改在将会生效，即修改操作发生后，再进行验证，参与验证的将是修改后的值。  
但是另一方面来看，这也是设计上的缺陷（code/design smell），更优雅的方式应该是在函数调用时传入不可变的对象，这样也会避免隐藏的bug，例如：被调用函数并未立即使用参数，而是在回调中/异步线程中使用参数，因此即便函数调用是同步进行的，后续的修改也会导致被调用函数使用参数时值发生了变化。  
更好的方式是这样的：
```java
Map map = new HashMap();
map.put("key", "value")
service.doSomething(map);
Map mapTwo = new HashMap();
mapTwo.put("key2", "value2");
serviceTwo.doSomethingElse(map);
```
另外，使用AutoValue/AutoParcel可以很方便的创建不可变的对象，但是在使用过程中还是容易“入坑”，例如使用了Collection类，即便元素对象是不可变的，但是collection并不是，如果按照上面的方式去实现，依然会导致问题，一方面，新new一个Map是一种解决方式，~~另一方面，如果是使用List，可以通过变长参数的方式来传递，这样就能避免这一问题~~，变长参数使用时仍然可能会有问题，例如：调用时传递的是一个数组对象，而非手动传递多个参数，那么如果多次调用之间传递的是同一个数组对象，那还是存在上面的问题，所以，无需变成传递变长参数，而是在调用时保证之后不再修改参数对象（TODO：go语言中有把数组打散之后传递的语法，是否能避免此问题？）。  

参考: [Google网上论坛](https://groups.google.com/d/msg/mockito/KBRocVedYT0/T-vgvqwjh0QJ)