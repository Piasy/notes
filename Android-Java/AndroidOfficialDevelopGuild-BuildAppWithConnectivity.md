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
