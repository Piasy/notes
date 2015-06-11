#Rx在Android中的最佳实践

+  By default, RxJava is synchronous  
	测试起来更方便；使用`Observable.from`创建observable，所有的subscription将立即触发；不要假设接收的subscription是异步的；
+  Hot and cold subscriptions  
	通常，只有observable被订阅（subscribe）时，所有的操作才会被执行；根据不同的实现，有可能每次有新的subscriber都会新创建一个operation，也有可能并不会；`.cache`操作可以缓存吐出的数据，后面的subscriber都会收到同一个数据；
+  Use subjects when in trouble  
	subject既能接收数据，也能吐出数据；
+  Pay attention to the thread  
	[rx线程模型](http://www.grahamlea.com/2014/07/rxjava-threading-examples/)；但是把切换回主线程放到最后并不一定是最好的；
+  Read the RxJava wiki and look at the diagrams  
	[官方wiki](https://github.com/ReactiveX/RxJava/wiki)
+  Subscribing with Observer vs. Action  
	使用Action1时如果发生错误，将抛出异常；最好是使用一个公用的ErrorHandler；
+  Subscriptions leak memory  
	匿名的observable会持有外部类的强引用，有可能导致内存泄漏；好的办法是每次使用observable，都将最后的subscription都保存起来，然后在合适的时机集中unSubscribe，以避免内存泄漏；