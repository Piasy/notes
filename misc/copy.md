#深入Java深浅拷贝、immutable、unmodifiable

+  建议：函数调用的时候，调用方传给被调用方的参数，如果在调用之后还会被修改，那么调用方应该给被调用方传一个当时的拷贝，深拷贝，否则：
  +  可能被调用方是异步执行的，如果调用函数之后，参数发生了修改，那么被调用方执行的时候，看到的就是被修改之后的数据，这将导致严重、隐蔽、非必现的BUG，而这种BUG是最让人头疼的
  +  可能被调用方会修改传入的参数，这就导致函数执行完毕之后，调用方看到的数据发生了非预期的变化，这同样会导致严重、隐蔽的BUG
+  深拷贝：这里需要弄清楚深浅拷贝的区别，用“=”号给非基本类型赋值，均是浅拷贝，例如List，以下代码就是浅拷贝：
```java
	List<Integer> list1 = new ArrayList<>();
	list1.add(1);
	List<Integer> list2 = list1;
	list1.add(2);
```
  代码执行完毕之后，list2将包含整数1和2。  
  而以下代码则是深拷贝：
```java
	List<Integer> list1 = new ArrayList<>();
	list1.add(1);
	List<Integer> list2 = new ArrayList<>(list1);
	list1.add(2);
```
  代码执行完毕之后，list2将只包含整数1，不包含整数2。
+  Immutable对象：上述情形如果遇到immutable对象，即不可变对象，其实是不需要深拷贝的（不仅不需要，还应该杜绝拷贝，因为纯属浪费）。但是前提是对象是真正的immutable。反面例子为：
```java
	public class NonStrictlyImmutable {
		private final List<Integer> mList = new ArrayList<>();
		
		public List<Integer> getList() {
			return mList;
		}
	}
```
  mList成员设置为了`private final`，NonStrictlyImmutable对象实例化完成后mList所引用的实际对象也不可再被改变，然而mList这个List的元素确是可以改变的，`nonStrictlyImmutable.getList().add(1)`并不会报编译错误，而这一行代码却实实在在改变了nonStrictlyImmutable对象的值！  
  而如果`getList()`函数不直接返回`mList`引用，创建一个副本，或者使其不可被改变，则可以达到”严格意义上的“immutable。例如：
```java
	public class NonStrictlyImmutable {
		private final List<Integer> mList = new ArrayList<>();
		
		public List<Integer> getList() {
			// 以下两种方式都可以，各有优劣
			// return new ArrayList<>(mList);
			// return Collections.unmodifiableList(mList);
			
			return Collections.unmodifiableList(mList);
		}
	}
```
  上面两种方式各有优劣：前者允许对新获取到的副本进行修改操作而不会抛出异常，但会把底层数组数据创建多份；后者不会创建多份底层数组数据，但是如果对`getList()`返回的引用进行修改操作，将会抛出异常；见仁见智。
+  unmodifiable：unmodifiable不等于immutable，所以对于前面提到的建议做法，直接传入unmodifiable是不对的，因为这样只能阻止被调用方修改传入数据后导致调用方出错，并不能阻止调用方修改后导致被调用方出错。unmodifiable并未拷贝底层数组数据，而是实现了另外一个List的实现类，该类的修改操作均抛出异常。