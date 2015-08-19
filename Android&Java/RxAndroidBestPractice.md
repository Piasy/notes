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
+  [Loading data from multiple sources with RxJava](http://blog.danlew.net/2015/06/22/loading-data-from-multiple-sources-with-rxjava/)
  +  多个source，由快到慢
  +  concat/concatWith，根据参数的顺序，依次把每个observable发射的item拼接起来，形成一个新的序列；参数之间发射的item不会重叠，即第一个observable的所有item发射完之后，第二个observable才会开始发射；
  +  first/takeFirst，从序列中取出第一个item，可以加过滤条件；first在过滤时如果没有满足条件的item，将会抛出异常，而takeFirst不会抛异常，只会调用onCompleted；
  +  两者结合，当第一个满足条件的item取到之后，concat参数中后面的observable将不会被subscribe；如果参数observable都是cold的，那么后面observable的产生操作将不会被执行；