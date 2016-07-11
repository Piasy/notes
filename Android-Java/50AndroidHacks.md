# 《50 Android Hacks》一书

## `include`, `merge`, `ViewStub` 的使用
+ `include` 可以用 `merge` 作为根节点，这样合并时可以省去一层 layout
+ `tools:showIn` 可以在预览中显示完整的 layout
+ `ViewStub` 可以懒惰加载 layout

## 获取 view 的高度和宽度
+ 可以在 `Activity.onCreate` 中 post 一个 runnable 到 view 上，runnable 执行时，view 的正确尺寸就可以获取到了

## 利用 ProGuard 移除 log 代码

```
# remove log call
-assumenosideeffects class android.util.Log {
    public static *** d(...);
}
-assumenosideeffects class timber.log.Timber {
    public static *** d(...);
}
```
