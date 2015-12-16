# Building Apps with Connectivity & the Cloud

## Connecting Devices Wirelessly
+  [Using Network Service Discovery，局域网下的P2P框架](http://developer.android.com/intl/zh-cn/training/connect-devices-wirelessly/nsd.html)
  +  自己可以提供服务，然后通过NSD框架暴露在本地局域网（注册服务）：`mNsdManager.registerService`；
  +  利用NSD框架，发现本地局域网的服务提供者：`mNsdManager.discoverServices`；
  +  解析发现的服务：`mNsdManager.resolveService`，解析成功的回调中，`NsdServiceInfo`参数携带了服务提供者（server）的信息，供建立连接；
  +  根据解析的结果（协议，地址），自行建立连接，进行通信即可
  +  当app退出，activity退出（onPause, onDestroy）时，需要将NSD服务取消注册
