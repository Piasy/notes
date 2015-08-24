#Memory leak专题
+  神器：[LeakCanary](https://github.com/square/leakcanary)，memory leak检测工具；

##Hanlder、Runnable、Thread的非静态内部类、匿名类，都会持有外部类的强引用，都可能造成内存泄漏；

##[Prior to Android Lollipop, alert dialogs may cause memory leaks in your Android apps.](https://corner.squareup.com/2015/08/a-small-leak.html)
+  考虑以下代码  
```java
while (true) {
    MyMessage msg = queue.take(); // might block
    System.out.println("Received: " + msg);
  }
```
msg对象是栈上的局部变量，每次循环都将会重写，一旦被重写，上一次循环的msg引用指向的对象将不再被其引用；但是在Dalvik虚拟机的实现中，如果queue.take()阻塞了，那么本次循环的msg未被赋值，则上次的msg的引用将不会被清除，
+  HandlerThread  
```java
for (;;) {
    Message msg = queue.next(); // might block
    if (msg == null) {
      return;
    }
    msg.target.dispatchMessage(msg);
    msg.recycleUnchecked();
  }
```
msg每次循环的后面都被recycle了（清空了内容），所以泄漏的仅仅是一个空的msg对象，影响不大（LeakCanary将默认忽略Message对象的泄漏）。
+  遇上AlertDialog  
```java
new AlertDialog.Builder(this)
    .setPositiveButton("Baguette", new DialogInterface.OnClickListener() {
      @Override public void onClick(DialogInterface dialog, int which) {
        MyActivity.this.makeBread();
      }
    })
    .show();
```
DialogInterface.OnClickListener的匿名实现类持有了MainActivity的强引用；而在AlertDialog的实现中，OnClickListener类将被包装在一个Message对象中，而且这个Message会在其内部被复制一份，两份Message中只有一个被recycle，另一个（OnClickListener的成员变量引用的Message对象）将会leak！
+  解决办法
  +  ART VM（>=5.0），JVM不存在此问题
  +  Message对象的泄漏无法避免，但是如果仅仅是一个空的Message对象，而且将被放入对象池作为后用，是没有问题的
  +  `DialogInterface.OnClickListener`对象不持有外部类的强引用：static类实现；DetachableClickListener（监听窗口解除事件，手动释放引用）；
  +  当worker thread空闲后，向HandlerThread发送一个空的消息，解除上一个Message的泄漏