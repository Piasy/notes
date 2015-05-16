#使用Style修饰View
+  当多个View语义上相同时：不仅样式相同，而且View的功能也相同；
  
+  当某项数值在多处使用时，style中的数值使用引用方式定义（抽离为dimen，color等）；
  +  将数值抽离出来，使用引用方式，不仅可以减小重复，方便修改；还能根据不同屏幕尺寸、横竖屏，为同一引用定义不同的值；
  
+  一个View不能使用多个style，但可以通过继承style，并且改变parent style的值，达到和CSS类似效果；
  
+  尽可能使用`android:textAppearance`属性，来定义`TextView`及其子类（`EditText`，`Button`等）的文字样式；

+  只会使用一次的时候，不要使用style；当需要重复使用的时候，添加style很方便，不必过度提前考虑；

+  不要只因为多个View使用同样的attributes就创建style；因为如果功能不同，有可能以后会变得样式不一样；

+  style指定parent有两种方式：隐式，`name="Parent.Child"`；显式，`name="Child" parent="Parent"`；
  +  不要两者同时使用，否则只有显式会生效；
  +  建议`name`属性不要带点，使用`parent`属性指定父style，而`parent`属性中使用点，隐式指定父style；

+  不要混淆style和theme；
  +  style修饰单个View，theme修饰View集合，或整个Activity；