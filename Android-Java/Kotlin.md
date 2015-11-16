# Kotlin学习笔记

## Basic Syntax
+  可见性  
Classes, objects, interfaces, constructors, functions, properties and their setters can have visibility modifiers. 
  +  private：包及子包私有；`foo.bar`为`foo`的子包
  +  protected：仅用于Class/Interface；可见性与private几乎一样，但是子类也是可见的
  +  internal：默认值；visible everywhere within the same module(?)
  +  public：visible everywhere
+  包管理
  +  声明的包路径不需要与文件系统路径一致
  +  private的可见性是包及子包私有
  +  父包的内容需要import才能访问
+  定义函数
  +  类似于go的语法  
  ```kotlin
  fun sum(a: Int, b: Int): Int {
    return a + b
  }
  ```
  +  当函数体只是返回一个表达式的值得时候，可以简化为：`public fun sum(a: Int, b: Int): Int = a + b`
  +  非public/protected时，返回类型可以省略
  +  不返回值时，返回类型可定义为Unit，Unit可以省略（public也可省略）
  +  Infix notation：当方法为成员方法，且只有一个参数时，可以简写为`1 shl 2`，等同于`1.shl(2)`
  +  可以定义默认参数，默认参数不必须在参数列表后面，但是调用的时候需要结合命名参数功能
  +  命名参数：调用函数的时候，可以通过参数名为其赋值，可以不按函数声明的参数顺序传参；结合默认参数，可以使得函数调用非常简洁
  +  函数体有block（分支、跳转）时，不支持返回类型推导
  +  vararg支持：函数的最后一个参数可以通过vararg修饰，达到var args的目的`fun asList<T>(vararg ts: T): List<T> {...}`
  
## To read
+  http://antonioleiva.com/kotlin/
+  http://antonioleiva.com/collection-operations-kotlin/
+  http://antonioleiva.com/plaid-kotlin-1/