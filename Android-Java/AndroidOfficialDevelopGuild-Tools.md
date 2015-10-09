#Developer tools

##Allocation tracker使用
+  用于记录app在profile周期中所有的内存分配，包括对象的调用栈、大小、分配代码
+  查找短周期内相同对象在相似代码位置的创建与销毁
+  查找代码中可能导致内存使用效率低的位置
+  使用过程
  +  打开工程，编译安装运行
  +  Android Studio的Android Monitor选项卡中，Memory tab，点击Start Allocation Tracking
  +  在需要测试的场景中与应用交互
  +  交互完成后，再次点击Start Allocation Tracking
  +  .alloc文件将在一会儿之后传输到工程根目录下的captures文件夹中，有可能传输会需要较长时间，需要耐心等待
