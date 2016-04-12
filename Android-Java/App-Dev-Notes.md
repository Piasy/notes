# 《App研发录》一书

## 第1章：重构
+ BaseActivity要有两个，一个是业务无关的，位于base module中，另一个是业务相关的，位于app module中，后者封装了业务相关的公用逻辑和代码；同理，当fragment, dialog fragment有业务相关的共性时，也就是时候为app module准备一个base类了；
+ package by feature，被外部引用的类，就不要作为内部类了；未被包外引用的类，就要声明为package private；
+ （如果）activity只用于管理fragment，fragment inflate view，绑定view的生命周期需要拆分开来，避免onViewCreated过于复杂；
+ 绑定view和初始化变量、调用获取数据的方法要拆分开来，单一职责！
  + 具体而言，应该有`getLayoutRes`, `initFields`, `bindView`, `startBusiness`, `unbindView`这5个方法
  + `getLayoutRes`：返回layout的id，用于onCreateView；
  + `initFields`, `bindView`, `startBusiness`：初始化成员变量、绑定view（包括设置listener）、开始执行业务逻辑，被onViewCreated**依次**调用；
  + `unbindView`：解绑view（包括重置listener）；
+ 为view设置listener，不要设置为this，而是创建新的；listener对象的创建开销并不是瓶颈，而代码整洁度的提升效果是很明显的；
+ fragment间数据传递：[FragmentArgs](https://github.com/sockeqwe/fragmentargs)；Effective Java item 74：尽量不要使用serialisable接口；
+ Adapter模板：建议使用[AdapterDelegates](https://github.com/sockeqwe/AdapterDelegates)，Favor composition over inheritance；而对于具体的一个AdapterDelegate，主要规范点在于onBindViewHolder，子view的listener如何设置，事件如何传递到fragment中？
  + 我的做法是，ViewHolder在构造函数中设置listener，同时它构造时接受一个接口，这个接口从fragment => adapter => ViewHolder => listener，如此达到事件传回fragment的目的；
+ 实体化编程/immutable/parcelable，推荐使用[Auto-parcel](https://github.com/frankiesardo/auto-parcel) + [Gson](https://github.com/google/gson)；
