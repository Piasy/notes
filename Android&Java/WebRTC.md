#WebRTC

##目标
+  便捷的为web应用添加视频聊天、p2p数据传输功能；
+  API开源、免费、标准化，嵌入到浏览器中，并且比现有插件技术更高效；

##API
+  [MediaStream (aka getUserMedia)](http://www.html5rocks.com/en/tutorials/webrtc/basics/#toc-mediastream)
+  [RTCPeerConnection](http://www.html5rocks.com/en/tutorials/webrtc/basics/#toc-rtcpeerconnection)
+  [RTCDataChannel](http://www.html5rocks.com/en/tutorials/webrtc/basics/#toc-rtcdatachannel)

+  MediaStream  
	具有input和output，input获取视频数据、音频数据；output将数据展示到video/audio标签，或发送到peer；
	```javascript
	var stream = navigator.getUserMedia()
	stream.getAudioTracks()
	stream.getVideoTracks()
	
	URL.createObjectURL() 
	```
+  Constraints  
	`applyConstraints()`函数为getUserMedia设置限制条件，支持分辨率、长宽比、前后摄像头、帧率、宽高等；  
	在一个标签页中设置了Constraints之后，后面打开的标签页都受影响；  
+  Screen and tab capture
	chrome目前支持屏幕录制、浏览器标签页录制API；
+  Signaling: session control, network and media information  
	WebRTC使用`RTCPeerConnection`API进行peer间通讯，`signaling`进程负责维护通信、发送控制信息；  
	+  Session control messages: to initialize or close communication and report errors.
	+  Network configuration: to the outside world, what's my computer's IP address and port?
	+  Media capabilities: what codecs and resolutions can be handled by my browser and the browser it wants to communicate with?
	
##[WebRTC on Android](https://tech.appear.in/2015/05/25/Introduction-to-WebRTC-on-Android/)