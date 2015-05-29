#Android项目架构
从功能需求、设计模式、最佳实践出发考虑

##MVP模式
数据、显示、控制解耦，[mosby](https://github.com/sockeqwe/mosby)，作者[blog1](http://hannesdorfmann.com/android/mosby/)，[blog2](http://hannesdorfmann.com/android/mosby-playbook/)

##依赖注入
+  普通数据注入，[Dagger](http://google.github.io/dagger/)
+  View注入[ButterKnife](http://jakewharton.github.io/butterknife/)。

##数据存储ORM
+  [squidb](https://github.com/yahoo/squidb)，基于注解，编译期生成DAO类，对复杂SQL语句的ORM非常好
+  ~~[DBFlow](https://github.com/Raizlabs/DBFlow)，基于注解，编译期生成DAO类，对关系的支持很好~~
+  ~~[ActiveAndroid](https://github.com/pardom/ActiveAndroid)，基于注解，运行时转换~~
+  ~~[greenDAO](https://github.com/greenrobot/greenDAO)，编译期生成辅助代码~~

##网络连接处理
+  [Retrofit](http://square.github.io/retrofit/)，使用动态代理将java接口转化为REST API
+  [OkHttp](http://square.github.io/okhttp/)，优化的HTTP client（共享连接，连接池，透明压缩，缓存，重试等），支持web socket协议

##异步处理
+  响应式编程（[rx](https://github.com/ReactiveX/RxAndroid)，在java7中使用lambda语法：[retrolambda](https://github.com/orfjackal/retrolambda)），消息的发出者和响应的接收者在同一模块内，局部性、单对单
+  事件处理（~~[EventBus](https://github.com/greenrobot/EventBus)~~，[Otto](http://square.github.io/otto/)），全局性事件、一对多比较合适

##测试
+  单元测试（[我的印象笔记](https://www.evernote.com/shard/s425/sh/ed5e5a9b-8ebf-4d72-8d4d-62bff3b57335/a172972c726caab6)，[Robolectric](http://robolectric.org/)，使得Android代码能够在PC的JVM上运行测试，加快速度）
+  集成测试（[我的印象笔记](https://www.evernote.com/shard/s425/sh/52ec6ce5-68ca-47d7-8fa1-14fdacfc3f1a/31fceed7211d8e13)，[Espresso](https://code.google.com/p/android-test-kit/wiki/Espresso)）

##持续集成
+  [jenkins-ci](http://jenkins-ci.org/)，整合代码托管工具，ci、mr自动触发构建

##错误统计
+  [fabric](https://get.fabric.io/)，提供crash统计、以及twitter集成

##图片加载
+  [Fresco](https://github.com/facebook/fresco)，多来源加载、缓存、内存管理、支持多种格式
+  ~~[Picasso](http://square.github.io/picasso/)，使用堆内存，格式稍少~~