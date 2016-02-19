# 《深入理解Android 5源代码》一书

## Java Native Interface(JNI)系统
+  JNI函数注册：把Java中声明的native方法与C/C++代码一一对应起来
  +  `JNINativeMethod`结构体，name, signature, fnPtr三个成员
  +  signature部分：V - void - void, Z - jboolean - boolean, J - jlong - long, 
  S - jshort - short, Ljava/lang/String; - jstring - String, 
  Ljava/net/Socket; - jobject - Socket
  +  通过在C/C++代码中设置好Java native方法对应的结构体数据，就可以达到将其对应起来的目的
  +  所有的注册工作都在`frameworks/base/core/jni/AndroidRuntime.cpp`中完成
  +  `AndroidRuntime::registerNativeMethods`函数调用了`JNIHelp.cpp`中的
  `jniRegisterNativeMethods`方法，=> `C_JNIEnv::RegisterNatives`
  +  如果不通过上述方式注册native函数，函数被调用时AndroidRuntime会从所有已载入的so库中
  搜索被调用的函数，效率较低，而上述注册会加速native函数的查找，只需在JNINativeMethod数组
  中查找即可
  +  而且由于上述注册机制，使得可以在运行时动态注册/hook
  +  动态注册
    +  Java层调用`System.loadLibrary`加载so库
    +  查找so库中的`JNI_OnLoad`函数，如果存在，则执行之
    +  可以在JNI_OnLoad函数中进行动态注册
  
## Hardware Abstract Layer(HAL)
+  将硬件抽象化，为操作系统提供虚拟硬件平台，操作系统的代码和硬件驱动的代码完全分离，软硬件完全分离，
便于保护硬件厂商的知识产权，也便于软硬件测试并行化
+  主要包括`hw_module_t`, `hw_module_methods_t`, `hw_device_t`
+  `hw_module_methods_t`为所有硬件操作定义了统一的API
+  硬件模块的编写有一定规范，HAL按照该规范载入硬件模块的SO库，获取绑定其中的符号、代码
