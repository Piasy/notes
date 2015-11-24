# React基础

## [React设计哲学](http://www.infoq.com/cn/articles/react-art-of-simplity/)
+  编写可预测，符合习惯的代码：代码简单易懂，易于维护
+  使用JSX直观的定义用户界面：传统方式，即便V与M文件分离，其逻辑依然是紧密关联的，所以干脆将它们文件上放到一起
+  简化的组件模型：所谓组件，其实就是状态机器；除了状态，组件还有属性；UI与状态绝对一致，修改UI仅仅修改状态即可；
+  每一次界面变化都是整体刷新：简化UI更新逻辑，由framework负责实际高效局部刷新；
+  单向数据流动：Flux，永远只有从模型到视图的数据流动
+  让数据模型也变简单：Immutability
+  React思想的衍生：React Native, React Canvas等等

## [React开发神器Webpack](http://www.infoq.com/cn/articles/react-and-webpack)
+  同时支持CommonJS和AMD模块
+  串联式模块加载器以及插件机制，让其具有更好的灵活性和扩展性
+  可以基于配置或者智能分析打包成多个文件，实现公共模块或者按需加载
+  支持对CSS，图片等资源进行打包，从而无需借助Grunt或Gulp
+  开发时在内存中完成打包，性能更快，完全可以支持开发过程的实时打包需求
+  对sourcemap有很好的支持，易于调试

## [理解JSX和组件](http://www.infoq.com/cn/articles/react-jsx-and-component)
