# Building Apps with Connectivity & the Cloud

## Connecting Devices Wirelessly
+  [Using Network Service Discovery，局域网下的服务提供、发现框架](http://developer.android.com/intl/zh-cn/training/connect-devices-wirelessly/nsd.html)
  +  自己可以提供服务，然后通过NSD框架暴露在本地局域网（注册服务）：`mNsdManager.registerService`；
  +  利用NSD框架，发现本地局域网的服务提供者：`mNsdManager.discoverServices`；
  +  解析发现的服务：`mNsdManager.resolveService`，解析成功的回调中，`NsdServiceInfo`参数携带了服务提供者（server）的信息，供建立连接；
  +  根据解析的结果（协议，地址），自行建立连接，进行通信即可
  +  当app退出，activity退出（onPause, onDestroy）时，需要将NSD服务取消注册
+  [利用Wi-Fi建立P2P连接](http://developer.android.com/intl/zh-cn/training/connect-devices-wirelessly/wifi-direct.html)
  +  声明权限：`ACCESS_WIFI_STATE`, `CHANGE_WIFI_STATE`, `INTERNET`
  +  监听系统广播事件：
    +  定义一个BroadcastReceiver，对收到的WifiP2p广播（wifi p2p状态变化、peer列表变化、p2p连接变化、peer信息变化等）进行响应
    +  在activity中注册BroadcastReceiver
  +  发现设备：`mManager.discoverPeers`，`mManager`为`WifiP2pManager`对象，通过`getSystemService`获得
  +  在BroadcastReceiver收到peer列表变化广播后，获取peer列表：`mManager.requestPeers`
  +  建立p2p连接：`mManager.connect`
  +  建立连接成功后，BroadcastReceiver会收到连接状态变化的广播，此时可以获取peer的详细地址信息，供建立socket连接用：`mManager.requestConnectionInfo`
+  [使用Wi-Fi P2P进行服务发现](http://developer.android.com/intl/zh-cn/training/connect-devices-wirelessly/nsd-wifi-direct.html)
  +  NSD框架需要设备处于同一局域网下，而Wi-Fi P2P则无需设备接入网络
  +  声明权限：`ACCESS_WIFI_STATE`, `CHANGE_WIFI_STATE`, `INTERNET`
  +  注册自己提供的服务：`WifiP2pDnsSdServiceInfo.newInstance`，`mManager.addLocalService`
  +  发现附近的服务提供者
    +  `WifiP2pManager.DnsSdTxtRecordListener`
    +  `WifiP2pManager.DnsSdServiceResponseListener`
    +  `mManager.setDnsSdResponseListeners(channel, dnsSdServiceResponseListener, dnsSdTxtRecordListener)`
    +  `mManager.addServiceRequest`
    +  `mManager.discoverServices`

## Performing Network Operations
+  进行网络通信
  +  大部分都采用HTTP协议，选择HTTP client很重要，[OkHttp](https://github.com/square/okhttp)已被安卓6.0作为系统默认HTTP client
  +  进行网络连接之前，需呀检查一下网络是否可用（仅仅是检查是否接入网络，并不能检测是否可以访问互联网）：`ConnectivityManager.getNetworkInfo(type)`，`ConnectivityManager.getActiveNetworkInfo()`和`NetworkInfo.isConnected()`
  +  也可以把网络访问出错的处理集中起来，总之，要么所有网络访问之前都进行网络检查，要么网络访问出错之后集中处理，这样代码更简洁优雅
  +  网络访问不能在UI线程上执行，使用[RxAndroid](https://github.com/ReactiveX/RxAndroid)可以方便的进行异步操作
+  管理网络使用
  +  通常，为用户提供以下设置选项，比较好：同步频率，是否仅在wifi下联网等
  +  监听网络状态变化：使用一个监听`ConnectivityManager.CONNECTIVITY_ACTION`广播的`BroadcastReceiver`，接收网络状态变化事件
  +  在代码中注册/反注册`BroadcastReceiver`，可以控制其活跃期
  +  如果在manifest中声明，则其活跃期将是系统启动到系统关闭，此时可以通过`PackageManager.setComponentEnabledSetting(...)`方法启用/禁用组件
+  解析XML
  +  安卓系统内置有xml解析器：[`XmlPullParser`](http://developer.android.com/intl/zh-cn/reference/org/xmlpull/v1/XmlPullParser.html)

## Transferring Data Without Draining the Battery
+  典型手机蜂窝网络状态机

  ![典型手机蜂窝网络状态机](../assets/mobile_radio_state_machine.png)

+  APP的网络请求应该考虑蜂窝网络的特性：积攒多个请求，集中发送；适当预取；积攒多次请求的数据，但是使用同一次请求批量发送；
+  使用DDMS/AndroidStudio的网络监控工具，可以查看APP网络请求的特征，并进行适当优化
+  进行数据备份、数据同步时，可以把网络请求交给GSM，它会将任务延迟至充电时、连接wifi时、或者是进行上述打包操作
+  适当缓存，以减小网络数据的传输
+  调整网络请求的模式，wifi、高带宽时，集中请求大量数据，减少网络请求的频率和次数
+  更多关于[电量优化](http://developer.android.com/training/monitoring-device-state/index.html)的内容，还需要在有实际开发需求的时候，进行实践和深入

## Backing up App Data to the Cloud
+  安卓系统提供了自动同步的功能，不过是同步到google drive，每个app拥有25MB的空间
+  同步有较好的开始策略（充电，连接wifi，处于空闲状态，或者24小时未同步），但是当25MB空间用完之后，没有备份替换策略，仅仅是停止备份
+  API 23起，系统默认备份APP的所有数据，可以在manifest中进行添加或排除，通过`android:fullBackupContent`指定配置文件
+  API 23之前的版本，系统提供了`BackupAgent`接口，同样也是备份到google服务器
+  google还提供了[Cloud Save service](http://developers.google.com/games/services/common/concepts/cloudsave)，用于保存数据（不是备份应用数据），同一用户在不同设备保存数据时，会有冲突，Cloud Save service也提供了相应的冲突处理接口

## Transferring Data Using Sync Adapters
+  安卓系统提供了一个用于和自己服务器同步数据的框架：Sync Adapter
+  使用该框架可以享受系统的调度服务，提升电池性能，提升网络访问性能；该框架是异步的，对于实时同步需求，无法使用此框架；
+  先创建一个`AbstractAccountAuthenticator`子类，以及一个`Service`用于连接`AccountAuthenticator`和Sync adapter框架，authenticator用于和自己的服务器进行认证，即便不需要认证，也需要实现一个空的authenticator
+  为`AccountAuthenticator`创建一个metadata xml描述文件，把`Service`声明到manifest中，并且使用`AccountAuthenticator`的metadata
+  sync adapter框架需要content provider来与之配合使用；如果不想使用content provider保存数据，也需要提供一个空的content provider实现，然后用自己的方式进行数据存储；
+  将content provider声明在manifest中
+  实现`AbstractThreadedSyncAdapter`子类，用于响应系统发出的同步通知，执行自己的数据同步逻辑
  +  在`onPerformSync()`中执行数据同步
  +  `onPerformSync()`在后台线程被调用，所以无需再次自行创建后台线程
  +  可以尝试在`onPerformSync()`中执行除了数据同步之外更多的网络请求，以集中网络请求，提示网络、电池效率
+  创建一个`Service`，用于连接`ThreadedSyncAdapter`和Sync adapter框架
+  和`AccountAuthenticator`类似，`ThreadedSyncAdapter`及其`Service`也需要定义metadata、声明到manifest文件中
+  在Activity中创建一个account，它将用于Sync adapter框架
+  `AccountAuthenticator`、`ThreadedSyncAdapter`都使用了binder机制，用于系统进程进行跨进程调用，获取自定义的AccountAuthenticator和ThreadedSyncAdapter实例，并执行其方法
+  sync adapter框架支持多种触发机制，高效不失灵活
  +  基于服务器的消息推送，表明服务器数据发生了变化
  +  本地数据变化事件触发（需要使用content provider）
  +  本地网络事件，由系统发送，自动进行
  +  定期执行
  +  定时执行
  +  按需执行
  
## Transmitting Network Data Using Volley
+  volly是一个HTTP网络请求的封装库，支持schedule，并发，两级缓存，优先级，取消，可定制等特性
+  volly执行流程

![../assets/volley-request.png](../assets/volley-request.png)

+  内置了`StringRequest`，`JsonObjectRequest`，`JsonArrayRequest`，`ImageRequest`请求类，用于把响应数据转换为目标数据格式
+  也可以自定义实现Request

## Building Apps with Location & Maps
+  Google play service提供的相关服务：automated location tracking, geofencing, and activity recognition
+  ...
