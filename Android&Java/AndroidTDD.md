#安卓测试驱动开发/安卓测试验证
安卓测试主要包括两种：单元测试；集成测试

##单元测试
单元测试旨在针对代码中关键方法/接口的逻辑正确性进行验证  
在非安卓系统组件相关的代码里面，进行单元测试比较好理解，也比较容易实现  
主要涉及的技术在于：单元测试框架；mock；系统组件相关的代码；
+  单元测试框架  
[Robolectric](http://robolectric.org/)，对于安卓系统相关的支持也比较全面，Resources，Toast，Database，Activity/Fragment生命周期
+  mock  
[Mockito](http://mockito.org/)，能mock对象，能验证被mock对象的方法被调用的次数，能为mock对象的函数插桩，设置被调用时的返回值
+  系统组件相关的代码  
主要测试部分：依赖于生命周期函数的逻辑；由简单UI交互引发的逻辑；自定义接口的逻辑（如MVP中的View）；
  +  依赖于生命周期函数的逻辑  
  需要保证在生命周期函数执行之前就能控制Fragment/Activity内的一些成员，使用mock对象  
  Robolectric对于系统组件生命周期的维护已经比较完善了；  
  ```java
	ActivityController controller = Robolectric.buildActivity(MyAwesomeActivity.class).create().start();
	Activity activity = controller.get();
	// assert that something hasn't happened
	activityController.resume();
	// assert it happened!
  ```
  Robolectric会维护系统组件内部的相关信息，当调用create的时候，就能确保Activity生命周期函数执行完了onCreate。如果想要触发其他函数，onPause，onResume等，也有相应的controller函数。同样支持模拟Intent启动，savedInstance恢复；  
  当需要通过Robolectric和View发生交互时，需要确保View都被附加到了window上，且已被显示，visible()方法的作用就在于此；
  
  ##[The Square Way](http://www.philosophicalhacker.com/2015/04/10/against-android-unit-tests/)
  架构图  
  ![androidstack-02.png](assets/androidstack-02.png)  
  Remove all business logic from app component classes (e.g., Activitys, Fragments, Services) and place that logic into “business objects”, POJO objects whose dependencies are injected, android-specific implementations of android-agnostic interfaces.  
  Delegate all application specific behavior to POJO objects whose dependencies are Android-specific implementations of Android-agnostic interfaces.
  +  Non-UI App Components  
  把业务逻辑（比如根据数据类型执行不同操作、检查数据合法性等）从组件类中抽离出来，业务逻辑类对于组件类的依赖也不要直接使用使用，而是通过定义接口来实现耦合，这样就能使得业务逻辑类与SDK解耦。  
  业务逻辑类的依赖通过依赖注入框架注入，便于测试时mock。  
  而剩下的组件类工作简单，只负责转发业务逻辑类的功能请求，就没必要进行测试了。
  +  UI App Component Classes  
  通过MVP模式，将UI组件类和业务逻辑类解耦，同时移除对SDK组件类的依赖；  
  pre-act-state，post-act-state，测试对象的依赖中，如果mock框架（如Mockito）可以mock，OK；如果不能mock（例如Activity，BroadcastReceiver），则可以通过定义接口的形式替换这些依赖，而接口的实现则是简单直接的转发，无需测试；pre-act-state便可以设置完毕，调用被测函数后，验证post-act-state即可。
  +  Dependency Inject  
  可以使用Constructor inject的类就不要使用Field inject。前者更利于单元测试。其实除了SDK组件类，其他的类基本上都可以使用Constructor inject。
  +  无需依赖第三方框架（Robolectric，Dagger）