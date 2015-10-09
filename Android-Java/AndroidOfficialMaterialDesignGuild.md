#Google官方Material Design手册（[developer](http://developer.android.com/design/)，[spec](http://www.google.com/design/spec)）

##Creative Vision
安卓系统、系统应用、谷歌核心应用，包括安卓开发者，都应牢记以下总体目标：
+  Enchant me：多层次的优美，流畅清晰的动画、松脆有意义的布局和排版、连图标都是一种艺术；应结合优美、简洁、功能目标，创造出使用简易但功能强大的APP；
+  Simplify my life：突出重点，简化操作；
+  Make me amazing：用新鲜技术打动用户，既简洁又优雅；

##Android Design Principles
+  Enchant Me
  +  Delight me in surprising ways：漂亮的界面、精心设计的动画、适时的音效...
  +  Real objects are more fun than buttons and menus：让用户可以直接接触、操作APP内的事物
  +  Let me make it mine：除了提供有价值、优美的默认外观/设置外，也适当提供自定义途径，当然，不能妨碍主要的功能
  +  Get to know me：记住用户的操作/设置/选择，尽量避免用户重复同样的选择，将以前的选择放在快捷的位置
+  Simplify My Life
  +  Keep it brief：内容描述尽量用短句，可读性才强
  +  Pictures are faster than words：图片更能吸引注意力，以及表达内容
  +  Decide for me but let me have the final say：尽最大努力猜测用户要执行的操作，先设置为默认值，而不是询问用户是否使用之；但要提供方便的取消功能，让用户拥有最终决定权
  +  Only show what I need when I need it：将内容分割为小块，方便接收；隐藏不重要的信息
  +  I should always know where I am：不同界面有明显区别，完善的导航，用动画暗示不同界面的关系，为执行中的任务提供反馈
  +  Never lose my stuff：记住用户的设置/创造，并能轻易访问，在不同设备之间提供同步功能
  +  If it looks the same, it should act the same：将功能的区别在视觉上明显体现出来，不要使用“模式设置”
  +  Only interrupt me if it's important：只在重要的时候提醒/打断用户
+  Make Me Amazing
  +  Give me tricks that work everywhere：使用一些通用的快捷操作/手势/标志
  +  It's not my fault：当用户操作不当时，提醒需要温和、易懂，掩盖技术细节，能自动修正就是最好的
  +  Sprinkle encouragement：将复杂操作拆分为简单的小任务，即便作用细微，也要为操作提供反馈
  +  Do the heavy lifting for me：让新手用得像专家一样
  +  Make important things fast：将最主要的功能放置在最突出的位置
  
##Material disign principles
+  Material is the metaphor：给用户真实事物的感受，真实触感、纸与墨、贴近真实物理规律、光影、表面、移动、相互关系等，都是真实世界的暗示；
+  Bold, graphic, intentional  
    The foundational elements of print-based design—typography, grids, space, scale, color, and use of imagery—guide visual treatments. These elements do far more than please the eye. They create hierarchy, meaning, and focus. Deliberate color choices, edge-to-edge imagery, large-scale typography, and intentional white space create a bold and graphic interface that immerse the user in the experience.  
    An emphasis on user actions makes core functionality immediately apparent and provides waypoints for the user.
+  Motion provides meaning：移动用来聚焦注意力、保持一致性；反馈很微妙；转变动画很有效；

##What is material?
+  Environment
  +  模拟3D世界，引入z坐标；
  +  模拟光影，结合key light和ambient light投射出的阴影；
+  Material properties
  +  物理属性
    +  只有宽高（xy）可以改变，厚度（z）固定为1dp；
    +  阴影体现的是元素的海拔高度（z），而不是颜色的变化；
    +  内容是显示在物质上的，而不是增加了物质的厚度；
    +  内容的行为独立于物质，但是受限于物质的边界；
    +  物质是实心的，输入事件（点击等）不能在物质之间传递，只能在最上层的物质上响应；
    +  多个物质元素不能同时在同一高度，要用阴影区分出它们的不同高度；
    +  物质不可穿过，改变高度时，不能穿过另一个物质；
  +  改变物质
    +  物质可以改变形状
    +  物质的变化都局限于其平面内
    +  物质不能折叠或者弯曲
    +  多个物质元素可以合并为同一个，只要在同一高度
    +  物质拆分、跨高度移动之后，又可以合并成一个
  +  物质的移动
    +  物质可以自发创建或销毁
    +  物质可以沿着任意轴移动
    +  沿着z轴移动通常暗示着和用户的交互
+  Objects in 3D space  
Material design中的元素和真实世界中的事物类似，可以叠起来，可以依附，但不能穿过，能投射阴影、反射光线；
  +  高度（Elevation）
    +  单位是dp，子元素的高度值是相对于父元素的，因为元素都是1dp厚，所以高度值是两个元素的顶部的距离；
    +  元素拥有默认的高度，App bar: 4dp，Floating Action Button: 6dp，Card: 2dp；
    +  默认高度不会改变，当改变一个元素的高度之后，应该尽快回到默认高度；
    +  同类元素的默认高度，在不同应用之间应该保持一致；但是在不同平台上可能不一致；
    +  响应式高度，改变高度以响应用户的输入（普通、聚焦、按下等），或者系统事件；
    +  动态高度变化，相对于默认高度，当需要响应时，变化一个偏移量，事件结束后回到默认高度；
    +  不同的阴影用来暗示不同的功能、区分不同的元素；
    +  阴影的变化可以用来暗示元素移动的方向；
  +  Object relationships
    +  元素之间的关系：父子关系；
    +  每个元素有一个父元素（系统，或者其他元素），每个元素可以有多个子元素；
    +  子元素从父元素继承属性，兄弟之间继承的属性是相同的；

##Animation
+  Authentic motion
动效可以用来表示元素之间的空间关系，功能，表现其优美与流畅。
  +  Mass and weight
    动效的开始、结束、改变方向，都应该和真实世界一样，需要一个过程；
    +  加速、减速过程要自然，最好用平滑的（加速度的变化是均匀的）、对称的加速减速过程；
    +  进入和离开时的加速度，要能反映出元素的意图，例如：减速就暗示着会停下来；移动的速度、方向变化，会引起用户的注意，要考虑在动效的过程中，哪一部分应该引起用户最大的注意力；
    +  不同大小的元素，加速度理应不一样，小的元素加速度应大一些；尽量使用[curved motion](http://developer.android.com/training/material/animations.html#CurvedMotion)；
+  Responsive interaction
  +  用户输入：点击、拖拽、声音、鼠标、键盘等，都应给出视觉上的响应；
  +  Surface reaction：使用ripple API，仅仅产生view表面的视觉变化；
  +  Material response：响应的动画要和触点有联系，如：动画的起止点都在触点处；产生和消失用大小动画；点击态高度增加；
  +  Radial action：点击事件需要有视觉反馈；点击触发的动画要和点击有联系，例如动画的起止点都在触点处；
+  Meaningful transitions  
  好的动效设计，可以有效帮助用户理解场景的切换、界面内元素的布局与变化、界面内元素的层级关系；
  +  Visual continuity  
    有助于引导用户的注意力；涉及到的元素：将进入场景的元素、将退出场景的元素、不同场景间共享的元素；相关阅读：[自定义Activity的切换动画](http://developer.android.com/training/material/animations.html#Transitions)；当设计动效时，需要考虑：引导用户的注意力、通过颜色和共享元素把动效在视觉上关联起来、精确地使用动效；
  +  Hierarchical timing  
    合理设计动效中元素之间的移动顺序、时间序列，要突出最重要的元素；
  +  Consistent choreography  
    合理设计移动的路径；相关阅读：[Use curved motion](http://developer.android.com/training/material/animations.html#CurvedMotion)；    
    最佳实践
    +  尽量避免线性空间路径，除非它就是需要这种限制；
    +  在不同的场景切换间，保持元素移动方向、路径的连贯性；避免冲突、交错的移动路径；
    +  元素移动时，它们处于哪一高度？为何这样设计？
    +  如果把每个元素的轨迹记录下来，形成的图像是否优美？是否清晰？
    +  使用连贯一致的进入、离开动效，强调元素之间的空间关系；
+  Delightful details  

##Style

##Layout

##Components

##Patterns

##Usability

##Resources

