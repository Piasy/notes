#Java同步机制

##synchronized关键字
+  有两种用法（或三种）
  +  synchronized方法
  +  synchronized代码块
    +  synchronized(this)
	  +  synchronized(object)
+  synchronized方法有两种效果
  +  对于同一个对象，多线程调用synchronized方法将只有一个线程能够进入执行，其他线程等待（不仅仅是对同一个方法来说，如果一个类的多个方法使用了synchronized修饰，记为func1, func2...，那多线程访问时，如果线程A正在访问func1，其他线程不仅访问func1会被阻塞，访问func2也会被阻塞）
  +  当正在执行的线程退出该方法时，其对对象状态（成员变量）造成的修改，将立即同步到其他线程中
+  synchronized(this)
  和synchronized方法有同样的效果，即synchronized方法，一个非synchronized但是整个方法体都是用synchronized(this)包括起来的方法，两者效果是等价的  
  即可以理解为synchronized方法其实就是方法体都用synchronized(this)所括起来的方法
+  synchronized(object)
  相当于把加锁对象由this改为object，对同一个object synchronized括起来的代码块，同时只能有一个线程执行
