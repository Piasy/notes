#Building Apps with Graphics & Animation

+  Displaying Bitmaps Efficiently
  +  手机内存资源有限，一个应用最少可能只有16MB的内存
  +  使用Bitmap显示图片，如果不压缩，很可能一个图片就会占用十几MB的内存
  +  ListView, GridView, ViewPager这种View，可能显示在屏幕上的只有几个图片，但是还有很多是没有显示在屏幕上的，如果没有显示在屏幕上的也占用着内存，那将很快导致OOM
  +  在解码/载入图片到内存（Bitmap对象）之前，要先判断一下图片的尺寸，获取图片的尺寸和类型：
  ```java
    BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
    int imageHeight = options.outHeight;
    int imageWidth = options.outWidth;
    String imageType = options.outMimeType;
  ```
  设置`options.inJustDecodeBounds = true`，将只读取文件信息，不会为图片分配内存
  +  载入和屏幕尺寸适配的图片尺寸，否则空占内存却没有视觉效果的提升，需要考虑以下因素：
    +  预计完全载入图片所需的内存大小
    +  预期为加载此图片愿意分配的内存
    +  用来显示该图片的View的尺寸
    +  当前设备的屏幕尺寸和像素密度
    ```java
        public static int calculateInSampleSize(
                    BitmapFactory.Options options, int reqWidth, int reqHeight) {
            // Raw height and width of image
            final int height = options.outHeight;
            final int width = options.outWidth;
            int inSampleSize = 1;
        
            if (height > reqHeight || width > reqWidth) {
        
                final int halfHeight = height / 2;
                final int halfWidth = width / 2;
        
                // Calculate the largest inSampleSize value that is a power of 2 and keeps both
                // height and width larger than the requested height and width.
                while ((halfHeight / inSampleSize) > reqHeight
                        && (halfWidth / inSampleSize) > reqWidth) {
                    inSampleSize *= 2;
                }
            }
        
            return inSampleSize;
        }
        
        public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
                int reqWidth, int reqHeight) {
        
            // First decode with inJustDecodeBounds=true to check dimensions
            final BitmapFactory.Options options = new BitmapFactory.Options();
            options.inJustDecodeBounds = true;
            BitmapFactory.decodeResource(res, resId, options);
        
            // Calculate inSampleSize
            options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
        
            // Decode bitmap with inSampleSize set
            options.inJustDecodeBounds = false;
            return BitmapFactory.decodeResource(res, resId, options);
        }
        
        ...
        mImageView.setImageBitmap(
            decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));
        ...
    ```
  +  加载、处理图片要在非UI线程执行，使用rx可以方便地做到这一点
  +  缓存Bitmap
    +  内存缓存：LruCache，放入cache的要用SoftReference或者WeakReference，以免放入缓存的数据未被按时移出缓存，导致内存泄漏
    +  磁盘缓存：DiskLruCache，；当要缓存的数据经常被访问时，使用ContentProvider会更合适
    +  响应Configuration change：使用headless fragment，是实现方式之一
  +  管理bitmap内存
    +  API <= 10：当确定bitmap不会被使用之后，调用bitmap.recycle()
    +  API >= 11：`BitmapFactory.Options.inBitmap`，会使得bitmap mutable，不太建议使用
  +  显示bitmap，将资源加载到内存的过程要在非UI线程执行，避免程序未响应
+  Displaying Graphics with OpenGL ES
  +  
+  Animating Views Using Scenes and Transitions, API >= 19, [Animations backported to Android 4.0+. API compatible with Android 2.2+](https://github.com/andkulikov/Transitions-Everywhere)
  +  The Transitions Framework
    +  动效的意义：帮助用户理解app的行为，理解视觉变化的意义
    +  包含的功能
        +  Group-level animations：可以应用于整个View hierarchy
        +  Transition-based animation：基于property animation
        +  Built-in animations：内置了很多动效实现
        +  Resource file support：支持在xml中定义动效
        +  Lifecycle callbacks：具备生命周期回调，便于响应动效状态进行逻辑操作
    +  原理  
    ![transitions_diagram.png](assets/transitions_diagram.png)
    +  Scene
        +  记录View hierarchy的状态
        +  支持从xml创建/从ViewGroup对象创建
        +  通常不需要显式指定开始的Scene，默认是前一个ending Scene，或者当前屏幕上显示的view hierarchy
        +  Scene的转化都发生在同一个Scene root中
    +  Transition
        +  在动效框架中，通过把scene的变化转化为一系列的关键帧，动画的信息保存在Transition对象中，通过TransitionManager来播放动画
        +  可以为两个Scene创建动画，也可以为当前Scene的不同状态创建动画
        +  内部实现了常用的动效：fading, resizing...
    +  Limitations
        +  对SurfaceView应用动效可能不会正常显示，SurfaceView在非UI线程进行更新
        +  有些动效可能对TextureView无效
        +  对AdapterView（例如ListView）使用动效，可能会导致界面卡住
        +  对TextView使用resize动效时，文字可能会在动效完成之前先“弹”出去，所以避免对有文字的view使用resize动效
  +  Creating a Scene
    +  从xml创建  
    scene root
    ```xml
        <FrameLayout
            android:id="@+id/scene_root">
            <include layout="@layout/a_scene" />
        </FrameLayout>
    ```
    starting scene
    ```xml
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@+id/scene_container"
            android:orientation="vertical"
            android:layout_width="match_parent"
            android:layout_height="match_parent" >
            <TextView
                android:id="@+id/text_view1
                android:text="Text Line 1" />
            <TextView
                android:id="@+id/text_view2
                android:text="Text Line 2" />
        </LinearLayout>
    ```
    ending scene
    ```xml
        <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@+id/scene_container"
            android:orientation="vertical"
            android:layout_width="match_parent"
            android:layout_height="match_parent" >
            <TextView
                android:id="@+id/text_view2
                android:text="Text Line 2" />
            <TextView
                android:id="@+id/text_view1
                android:text="Text Line 1" />
        </LinearLayout>
    ```
    创建scene
    ```java
        Scene mAScene;
        Scene mAnotherScene;
        
        // Create the scene root for the scenes in this app
        mSceneRoot = (ViewGroup) findViewById(R.id.scene_root);
        
        // Create the scenes
        mAScene = Scene.getSceneForLayout(mSceneRoot, R.layout.a_scene, this);
        mAnotherScene =
            Scene.getSceneForLayout(mSceneRoot, R.layout.another_scene, this);
    ```
    +  代码创建
    ```java
        Scene mScene;
        
        // Obtain the scene root element
        mSceneRoot = (ViewGroup) mSomeLayoutElement;
        
        // Obtain the view hierarchy to add as a child of
        // the scene root when this scene is entered
        mViewHierarchy = (ViewGroup) someOtherLayoutElement;
        
        // Create a scene
        mScene = new Scene(mSceneRoot, mViewHierarchy);
    ```
    +  Create Scene Actions
        +  当涉及的View不在同一个view hierarchy中时，可以通过创建scene action来辅助实现动效
        +  动效框架无法自动创建scene action时，可以手动创建，例如ListView
        +  把要执行的操作封装到Runnable中，设置给`Scene.setExitAction()`和`Scene.setEnterAction()`
        +  注意：不要在scene action中传递数据
  +  应用动效
    +  内置动效
        +  AutoTransition/&lt;autoTransition/&gt;：Default transition. Fade out, move and resize, and fade in views, in that order.
        +  Fade/&lt;fade/&gt;：fade_in fades in views; fade_out fades out views; fade_in_out (default) does a fade_out followed by a fade_in.
        +  ChangeBounds/&lt;changeBounds/&gt;：Moves and resizes views.
    +  从xml创建  
    res/transition/fade_transition.xml
    ```xml
        <fade xmlns:android="http://schemas.android.com/apk/res/android" />
    ```
    inflate
    ```java
        Transition mFadeTransition =
                TransitionInflater.from(this).
                inflateTransition(R.transition.fade_transition);
    ```
    +  代码创建：`Transition mFadeTransition = new Fade();`
    +  应用动效：`TransitionManager.go(mEndingScene, mFadeTransition);`
    +  支持只对view hierarchy中的部分view应用动效：`removeTarget()`，`addTarget()`
    +  同时使用多种效果：TransitionSet  
    在res/transitions/目录下创建资源文件
    ```xml
        <transitionSet xmlns:android="http://schemas.android.com/apk/res/android"
            android:transitionOrdering="sequential">
            <fade android:fadingMode="fade_out" />
            <changeBounds />
            <fade android:fadingMode="fade_in" />
        </transitionSet>
    ```
    +  不使用Scene而直接应用动效
        +  通过removeView/addView来改变view hierarchy， 并使用delayed transition
        +  改变view hierarchy之前调用TransitionManager.beginDelayedTransition()
        +  修改view：增/减/改
    +  实现TransitionListener以在动效生命周期回调中执行操作
  +  自定义Transition
    +  继承Transition类
    ```java
        public class CustomTransition extends Transition {
        
            @Override
            public void captureStartValues(TransitionValues values) {}
        
            @Override
            public void captureEndValues(TransitionValues values) {}
        
            @Override
            public Animator createAnimator(ViewGroup sceneRoot,
                                        TransitionValues startValues,
                                        TransitionValues endValues) {}
        }
    ```
    +  Capture View Property Values
        +  `captureStartValues`，scene中的每个view都会调用一遍，TransitionValues包含view对象，和一个map，用于记录view对象的各个感兴趣的属性值
        ```java
            public class CustomTransition extends Transition {
            
                // Define a key for storing a property value in
                // TransitionValues.values with the syntax
                // package_name:transition_class:property_name to avoid collisions
                private static final String PROPNAME_BACKGROUND =
                        "com.example.android.customtransition:CustomTransition:background";
            
                @Override
                public void captureStartValues(TransitionValues transitionValues) {
                    // Call the convenience method captureValues
                    captureValues(transitionValues);
                }
            
            
                // For the view in transitionValues.view, get the values you
                // want and put them in transitionValues.values
                private void captureValues(TransitionValues transitionValues) {
                    // Get a reference to the view
                    View view = transitionValues.view;
                    // Store its background property in the values map
                    transitionValues.values.put(PROPNAME_BACKGROUND, view.getBackground());
                }
                ...
            }
        ```
        +  `captureEndValues`和`captureStartValues`类似
        ```java
            @Override
            public void captureEndValues(TransitionValues transitionValues) {
                captureValues(transitionValues);
            }
        ```
        +  `createAnimator`，transition framework调用此函数的次数与scene的变化有关，例如5个view，变化后有2个被移除，1个新加入，则会调用6次：3个保持存在，2个移除，1个加入；保持存在的view，调用时两个TransitionValues都不为null，而只存在于一个状态的，另一个状态对应的TransitionValues参数为null；最终利用两个TransitionValues参数，返回一个property animator
+  