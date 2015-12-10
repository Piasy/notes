# 安卓系统动效专题

## [系统API](https://developer.android.com/training/material/animations.html#Transitions)

## [入门博客系列](http://www.androiddesignpatterns.com/2014/12/activity-fragment-transitions-in-android-lollipop-part1.html)

## [Backport lib](https://github.com/andkulikov/Transitions-Everywhere)

## [自定义Activity的过场动效](https://www.youtube.com/watch?v=CPxkoe2MraA)

## [Fragment过场动效](http://stackoverflow.com/a/17488542/3077508)
```java
mFragmentManager.beginTransaction()
                .setCustomAnimations(R.anim.slide_down, R.anim.slide_up, R.anim.slide_down, R.anim.slide_up)
                .add(android.R.id.content, new SelfHomeFragment(), SelfHomeFragment.class.getName())
                .commit();
```
setCustomAnimations调用的顺序一定要是第一个，否则会不起效。  
使用add、remove，则会有两层Fragment同时显示的效果。

## [Fragment share animation](https://medium.com/@bherbst/fragment-transitions-with-shared-elements-7c7d71d31cbb)

## Transition Animations
+  property animator
  +  通过动画API改变view的属性（位置），动画结束后，改变view的位置；
  +  `Animator`、`ObjectAnimator.ofFloat(Object target, String propertyName, float... values)`、``
+  基于Scene的动效：系统自动检测两个Scene之间的区别，然后用动效进行过渡；
+  Scene可以用代码创建，也可以由xml定义；
+  `ChanngeBounds`，`Fade`，`AutoTransition`，...，`TransitionSet`，`AutoTransition.setOrdering()`，`TransitionSet.setOrdering()`；
+  变化的View要有公共的父Layout；
+  无公共父Layout：`setReparent()`，但是效果并不是十分完美；