#Kotlin学习笔记

##Basic Syntax
+  可见性  
Classes, objects, interfaces, constructors, functions, properties and their setters can have visibility modifiers. 
  +  private：包及子包私有；`foo.bar`为`foo`的子包
  +  protected：仅用于Class/Interface；可见性与private几乎一样，但是子类也是可见的
  +  internal：默认值；visible everywhere within the same module
  +  public：visible everywhere
+  包管理
  +  声明的包路径不需要与文件系统路径一致
  +  private的可见性是包及子包私有
  +  父包的内容需要import才能访问
+  定义函数