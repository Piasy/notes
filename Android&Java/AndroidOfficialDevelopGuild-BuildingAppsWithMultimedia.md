#[安卓官方开发指南](http://developer.android.com/training/index.html)

##Building Apps with Multimedia
+  Managing Audio Playback
  +  控制音量和播放
    +  Audio Stream：安卓系统为不同的用途维护了不同的audio stream，便于用户控制不同类型声音的音量
	  +  music
	  +  alarm
	  +  notification
	  +  来电话
	  +  system sound
	  +  打电话过程中的声音
	  +  DTMF tones
	+  通过设置`setVolumeControlStream()`，系统将在Activity/Fragment仍在界面上时，自动响应设备的音量操作键，增大或减小设置类型媒体的音量
	+  当用户通过耳机等外设，按下播放控制按键时，例如：播放/暂停、上一曲/下一曲，系统将发送一个action为`android.intent.action.MEDIA_BUTTON`的广播，如下BroadcastReceiver可以响应处理这一广播（需要在AndroidManifest.xml中声明）：
	```java
		public class RemoteControlReceiver extends BroadcastReceiver {
			@Override
			public void onReceive(Context context, Intent intent) {
				if (Intent.ACTION_MEDIA_BUTTON.equals(intent.getAction())) {
					KeyEvent event = (KeyEvent)intent.getParcelableExtra(Intent.EXTRA_KEY_EVENT);
					if (KeyEvent.KEYCODE_MEDIA_PLAY == event.getKeyCode()) {
						// Handle key press.
					}
				}
			}
		}
	```
  +  Managing Audio Focus
    +  请求/释放audio focus
	```java
		AudioManager am = mContext.getSystemService(Context.AUDIO_SERVICE);
		...
		
		// Request audio focus for playback
		int result = am.requestAudioFocus(afChangeListener,
										// Use the music stream.
										AudioManager.STREAM_MUSIC,
										// Request permanent focus.
										AudioManager.AUDIOFOCUS_GAIN);
		
		if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
			am.registerMediaButtonEventReceiver(RemoteControlReceiver);
			// Start playback.
		}
		
		...
		// Abandon audio focus when playback complete    
		am.abandonAudioFocus(afChangeListener);
	```
  +  `requestAudioFocus`的最后一个参数可以设为`AudioManager.AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK`，用于请求短暂的audio focus，允许其他app在失去audio focus时继续播放音乐（但应该降低音量）
  +  监听audio focus状态改变
  ```java
	OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
		public void onAudioFocusChange(int focusChange) {
			if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT
				// Pause playback
			} else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
				// Resume playback 
			} else if (focusChange == AudioManager.AUDIOFOCUS_LOSS) {
				am.unregisterMediaButtonEventReceiver(RemoteControlReceiver);
				am.abandonAudioFocus(afChangeListener);
				// Stop playback
			}
		}
	};
  ```
  +  响应临时失焦且允许重音的情形
  ```java
	OnAudioFocusChangeListener afChangeListener = new OnAudioFocusChangeListener() {
		public void onAudioFocusChange(int focusChange) {
			if (focusChange == AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK) {
				// Lower the volume
			} else if (focusChange == AudioManager.AUDIOFOCUS_GAIN) {
				// Raise it back to normal
			}
		}
	};
  ```
  +  检查声音播放设备
  ```java
	if (audioManager.isBluetoothA2dpOn()) {
		// A2DP audio routing to the Bluetooth headset
	} else if (audioManager.isSpeakerphoneOn()) {
		// Adjust output for Speakerphone.
	} else if (audioManager.isWiredHeadsetOn()) {
		// Adjust output for headsets
	} else if (audioManager.isBluetoothScoOn()) {
		// SCO is used for communications
	} else { 
		// If audio plays and noone can hear it, is it still playing?
	}
  ```
  +  一旦耳机/蓝牙耳机断开连接，系统将继续使用默认设备（扬声器）播放，同时系统会发送一个`AudioManager.ACTION_AUDIO_BECOMING_NOISY`广播，可以通过以下代码进行响应
  ```java
	private class BroadcastReceiver myNoisyAudioStreamReceiver = new BroadcastReceiver() {
		@Override
		public void onReceive(Context context, Intent intent) {
			if (AudioManager.ACTION_AUDIO_BECOMING_NOISY.equals(intent.getAction())) {
				// Pause the playback
			}
		}
	};
	
	private void startPlayback() {
		registerReceiver(myNoisyAudioStreamReceiver, new IntentFilter(AudioManager.ACTION_AUDIO_BECOMING_NOISY));
	}
	
	private void stopPlayback() {
		unregisterReceiver(myNoisyAudioStreamReceiver);
	}
  ```
+  Capturing Photos
  +  使用已有相机APP拍照
    +  声明使用相机的feature，注意，并非权限，便于google play等应用商店确定设备是否可以安装本应用
	```xml
		<manifest ... >
			<uses-feature android:name="android.hardware.camera" android:required="true" />
			...
		</manifest>
	```
	+  也可以设置required为false，手动检查设备是否有相机：`packageManager.hasSystemFeature(PackageManager.FEATURE_CAMERA)`
	+  发送Intent调起已有相机APP拍照，发送Intent之前需要检查是否有其他APP可以响应此Intent，如没有却调用了startActivity，将会抛出异常
	```java
		static final int REQUEST_IMAGE_CAPTURE = 1;
		
		private void dispatchTakePictureIntent() {
			Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
			if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
				startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
			}
		}
	```
	在发送takePictureIntent之前，可以手动设置要照片要保存的位置`takePictureIntent.putExtra(android.provider.MediaStore.EXTRA_OUTPUT, imageFileUri);`，拍照成功返回之后，可以直接访问该uri。
	+  获取拍照结果缩略图
	```java
		@Override
		protected void onActivityResult(int requestCode, int resultCode, Intent data) {
			if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
				Bundle extras = data.getExtras();
				Bitmap imageBitmap = (Bitmap) extras.get("data");
				mImageView.setImageBitmap(imageBitmap);
			}
		}
	```
	+  获取完整照片  
	照片保存在外置存储卡的公开区域，需要声明权限，`WRITE_EXTERNAL_STORAGE`包含了`READ_EXTERNAL_STORAGE`权限，目录路径通过`Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES)`函数获得
	```xml
		<manifest ...>
			<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
			...
		</manifest>
	```
	如果要保存在APP私有目录下，API 18之后，将不用声明该权限，目录路径通过`context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)`函数获得，app访问自己对应的该目录，从API 19起，将不需要任何权限，但是访问其他APP对应的目录时，需要`WRITE_EXTERNAL_STORAGE`/`READ_EXTERNAL_STORAGE`权限，该目录不一定任何时候都可以访问，也不具备安全性
	```xml
		<manifest ...>
			<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
							android:maxSdkVersion="18" />
			...
		</manifest>
	```
	拍照设置保存文件
	```java
		String mCurrentPhotoPath;
		
		private File createImageFile() throws IOException {
			// Create an image file name
			String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
			String imageFileName = "JPEG_" + timeStamp + "_";
			File storageDir = Environment.getExternalStoragePublicDirectory(
					Environment.DIRECTORY_PICTURES);
			File image = File.createTempFile(
				imageFileName,  /* prefix */
				".jpg",         /* suffix */
				storageDir      /* directory */
			);
		
			// Save a file: path for use with ACTION_VIEW intents
			mCurrentPhotoPath = "file:" + image.getAbsolutePath();
			return image;
		}
		
		static final int REQUEST_TAKE_PHOTO = 1;
		
		private void dispatchTakePictureIntent() {
			Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
			// Ensure that there's a camera activity to handle the intent
			if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
				// Create the File where the photo should go
				File photoFile = null;
				try {
					photoFile = createImageFile();
				} catch (IOException ex) {
					// Error occurred while creating the File
					...
				}
				// Continue only if the File was successfully created
				if (photoFile != null) {
					takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT,
							Uri.fromFile(photoFile));
					startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
				}
			}
		}
	```
	+  加入Gallery，当保存路径设为`context.getExternalFilesDir(type)`时，将无法加入Gallery
	```java
		private void galleryAddPic() {
			Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
			File f = new File(mCurrentPhotoPath);
			Uri contentUri = Uri.fromFile(f);
			mediaScanIntent.setData(contentUri);
			this.sendBroadcast(mediaScanIntent);
		}
	```
	+  获取压缩后的图片，用于显示在ImageView上
	```java
		private void setPic() {
			// Get the dimensions of the View
			int targetW = mImageView.getWidth();
			int targetH = mImageView.getHeight();
		
			// Get the dimensions of the bitmap
			BitmapFactory.Options bmOptions = new BitmapFactory.Options();
			bmOptions.inJustDecodeBounds = true;
			BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
			int photoW = bmOptions.outWidth;
			int photoH = bmOptions.outHeight;
		
			// Determine how much to scale down the image
			int scaleFactor = Math.min(photoW/targetW, photoH/targetH);
		
			// Decode the image file into a Bitmap sized to fill the View
			bmOptions.inJustDecodeBounds = false;
			bmOptions.inSampleSize = scaleFactor;
			bmOptions.inPurgeable = true;
		
			Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
			mImageView.setImageBitmap(bitmap);
		}
	```
  +  使用已有相机应用录制视频，与拍照类似，需要发送的Intent初始化为`Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);`，`onActivityResult`返回的intent的data数据，就是录制视频的Uri，`intent.getData()`。
  +  直接使用相机
    +  Camera API
	  +  权限
	  ```xml
		<uses-permission android:name="android.permission.CAMERA" />  
		<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
		<uses-feature android:name="android.hardware.camera" android:required="true" />
	  ```
	  +  在onResume中inflate SurfaceView，动态添加到layout中，在onPause中停止预览，移除SurfaceView，以解决界面onPause后再onResume就无法预览的问题。在SurfaceView的`surfaceCreated`回调中打开相机，开始预览
	  ```java
		@Override
		public void surfaceCreated(SurfaceHolder holder) {
			mSurfaceHolder = holder;
			initPreview(holder);
		}
	
		@Override
		public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
	
		}
	
		@Override
		public void surfaceDestroyed(SurfaceHolder holder) {
			releaseResources();
		}
		
		@Override  
    	protected void onResume() {  
            mSurfaceView = (SurfaceView) LayoutInflater.from(getActivity())
                    .inflate(R.layout.ui_surface_view, null);
            flContainer.addView(mSurfaceView, 0);
            isPreview = true;
            SurfaceHolder mSurfaceHolder = mSurfaceView.getHolder();
            mSurfaceHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
            mSurfaceHolder.setKeepScreenOn(true);
            mSurfaceHolder.addCallback(this);
		}
	  ```
	  +  打开相机，开始预览
	  ```java
		private boolean initPreview(SurfaceHolder holder) {
			int mSupportVideoFormat[] = {ImageFormat.NV21, ImageFormat.YV12};
			try {
				cameraid = getCameraId(isFrontCamera);
				mCamera = Camera.open(cameraid);
			} catch (Exception e) {
				e.printStackTrace();
				return false;
			}
	
			if (mCamera == null) {
				ToastUtils.toastResId(R.string.error_init_player_fail);
				return false;
			}
	
			// change to portrait record
			setCameraDisplayOrientation(cameraid, mCamera);
			try {
				mCamera.setPreviewDisplay(holder);
			} catch (IOException e) {
				ToastUtils.toastResId(R.string.error_IO_error);
				e.printStackTrace();
				return false;
			}
			Camera.Parameters parameters = mCamera.getParameters();
			parameters.setPreviewSize(width, height);
			parameters.setPictureSize(width, height);
			if (isLightOn) {
				parameters.setFlashMode(Camera.Parameters.FLASH_MODE_TORCH);
			}
	
			int actualFormat = 0;
			List<Integer> list = parameters.getSupportedPreviewFormats();
			for (int format : mSupportVideoFormat) {
				for (Integer i : list) {
					Timber.i("startVideoCapture " + "suport format:" + i);
					if (format == i.intValue()) {
						actualFormat = format;
						break;
					}
				}
				if (actualFormat != 0) {
					break;
				}
			}
			if (actualFormat == 0) {
				Timber.e("startVideoCapture" + " no suport format be found");
				return false;
			}
			parameters.setPreviewFormat(actualFormat);// ImageFormat.YV12
			try {
				mCamera.setParameters(parameters);
				mCamera.startPreview();
			} catch (Exception e) {
				e.printStackTrace();
				return false;
			}
	
			return true;
		}
	  ```
	+  [Google Camera2 Sample](https://github.com/googlesamples/android-Camera2Basic/blob/241c6fac81/Application%2Fsrc%2Fmain%2Fjava%2Fcom%2Fexample%2Fandroid%2Fcamera2basic%2FCamera2BasicFragment.java)，[简版](http://blog.csdn.net/torvalbill/article/details/40378539)
	  +  权限
	  ```xml
		<uses-sdk  
			android:minSdkVersion="21"  
			android:targetSdkVersion="21" />  
		<uses-permission android:name="android.permission.CAMERA" />  
		<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />  
		<uses-feature android:name="android.hardware.camera2.full" /> 
	  ```
	  +  layout
	  ```xml
		<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
			xmlns:tools="http://schemas.android.com/tools"
			android:layout_width="match_parent"
			android:layout_height="match_parent"
			android:paddingBottom="@dimen/activity_vertical_margin"
			android:paddingLeft="@dimen/activity_horizontal_margin"
			android:paddingRight="@dimen/activity_horizontal_margin"
			android:paddingTop="@dimen/activity_vertical_margin"
			tools:context="com.example.camera2te.MainActivity" >
		
			<TextureView
				android:id="@+id/texture"
				android:layout_width="match_parent"
				android:layout_height="match_parent"
				android:layout_alignParentStart="true"
				android:layout_alignParentTop="true" />
		</RelativeLayout>
	  ```
	  +  设置TextureView回调，在`onSurfaceTextureAvailable`回调中打开相机
	  ```java
		private TextureView.SurfaceTextureListener mSurfaceTextureListener = new TextureView.SurfaceTextureListener(){  
	
			@Override  
			public void onSurfaceTextureAvailable(SurfaceTexture surface, int width, int height) {  
				Log.e(TAG, "onSurfaceTextureAvailable, width="+width+",height="+height);  
				openCamera();  
			}  
	
			@Override  
			public void onSurfaceTextureSizeChanged(SurfaceTexture surface,  
					int width, int height) {  
				
			}  
	
			@Override  
			public boolean onSurfaceTextureDestroyed(SurfaceTexture surface) {  
				return false;  
			}  
	
			@Override  
			public void onSurfaceTextureUpdated(SurfaceTexture surface) {  
				
			}  
			
		};
		
		@Override  
    	protected void onResume() {  
			...  
			mTextureView.setSurfaceTextureListener(mSurfaceTextureListener);
			...
		}
	  ```
	  +  打开相机，在相机回调中开始预览
	  ```java
		private void openCamera() {  
			CameraManager manager = (CameraManager) getSystemService(Context.CAMERA_SERVICE); 
			try {  
				String cameraId = manager.getCameraIdList()[0];  
				CameraCharacteristics characteristics = manager.getCameraCharacteristics(cameraId);  
				StreamConfigurationMap map = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);  
				mPreviewSize = map.getOutputSizes(SurfaceTexture.class)[0];  
				
				manager.openCamera(cameraId, mStateCallback, null);  
			} catch (CameraAccessException e) {  
				e.printStackTrace();  
			}
		}  
		
		private CameraDevice.StateCallback mStateCallback = new CameraDevice.StateCallback() {  
	
			@Override  
			public void onOpened(CameraDevice camera) {    
				mCameraDevice = camera;  
				startPreview();  
			}  
	
			@Override  
			public void onDisconnected(CameraDevice camera) {  
				
			}  
	
			@Override  
			public void onError(CameraDevice camera, int error) {  
	
			}  
			
		};  
	  ```
	  +  开启预览
	  ```java
		protected void startPreview() {  
			if(null == mCameraDevice || !mTextureView.isAvailable() || null == mPreviewSize) {  
				Log.e(TAG, "startPreview fail, return");  
				return;  
			}  
			
			SurfaceTexture texture = mTextureView.getSurfaceTexture();  
			if(null == texture) {  
				Log.e(TAG,"texture is null, return");  
				return;  
			}  
			
			texture.setDefaultBufferSize(mPreviewSize.getWidth(), mPreviewSize.getHeight());  
			Surface surface = new Surface(texture);  
			
			try {  
				mPreviewBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);  
			} catch (CameraAccessException e) {  
	
				e.printStackTrace();  
			}  
			mPreviewBuilder.addTarget(surface);  
			
			try {  
				mCameraDevice.createCaptureSession(Arrays.asList(surface), new CameraCaptureSession.StateCallback() {  
					
					@Override  
					public void onConfigured(CameraCaptureSession session) {  
	
						mPreviewSession = session;  
						updatePreview();  
					}  
					
					@Override  
					public void onConfigureFailed(CameraCaptureSession session) {  
	
						Toast.makeText(MainActivity.this, "onConfigureFailed", Toast.LENGTH_LONG).show();  
					}  
				}, null);  
			} catch (CameraAccessException e) {  
	
				e.printStackTrace();  
			}  
		}  
		
		protected void updatePreview() {  
			if(null == mCameraDevice) {  
				Log.e(TAG, "updatePreview error, return");  
			}  
				
			mPreviewBuilder.set(CaptureRequest.CONTROL_MODE, CameraMetadata.CONTROL_MODE_AUTO);  
			HandlerThread thread = new HandlerThread("CameraPreview");  
			thread.start();  
			Handler backgroundHandler = new Handler(thread.getLooper());  
				
			try {  
				mPreviewSession.setRepeatingRequest(mPreviewBuilder.build(), null, backgroundHandler);  
			} catch (CameraAccessException e) {  
		
				e.printStackTrace();  
			}  
		}  
	  ```
+  Printing Content, >= API 19
  +  打印图片
  ```java
    PrintHelper photoPrinter = new PrintHelper(getActivity());
    photoPrinter.setScaleMode(PrintHelper.SCALE_MODE_FIT);
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(),
            R.drawable.droids);
    photoPrinter.printBitmap("droids.jpg - test print", bitmap);
  ```
  +  ScaleMode
    +  SCALE_MODE_FIT：等比例缩放图片，使之可以在打印区域内显示
	+  SCALE_MODE_FILL：充满打印区域，上下/左右可能会有部分内容无法打印
  +  打印HTML文档
  ```java
	private WebView mWebView;
	
	private void doWebViewPrint() {
		// Create a WebView object specifically for printing
		WebView webView = new WebView(getActivity());
		webView.setWebViewClient(new WebViewClient() {
	
				public boolean shouldOverrideUrlLoading(WebView view, String url) {
					return false;
				}
	
				@Override
				public void onPageFinished(WebView view, String url) {
					Log.i(TAG, "page finished loading " + url);
					createWebPrintJob(view);
					mWebView = null;
				}
		});
	
		// Generate an HTML document on the fly:
		String htmlDocument = "<html><body><h1>Test Content</h1><p>Testing, " +
				"testing, testing...</p></body></html>";
		webView.loadDataWithBaseURL(null, htmlDocument, "text/HTML", "UTF-8", null);
	
		// Keep a reference to WebView object until you pass the PrintDocumentAdapter
		// to the PrintManager
		mWebView = webView;
	}
	
	private void createWebPrintJob(WebView webView) {
	
		// Get a PrintManager instance
		PrintManager printManager = (PrintManager) getActivity()
				.getSystemService(Context.PRINT_SERVICE);
	
		// Get a print adapter instance
		PrintDocumentAdapter printAdapter = webView.createPrintDocumentAdapter();
	
		// Create a print job with name and adapter instance
		String jobName = getString(R.string.app_name) + " Document";
		PrintJob printJob = printManager.print(jobName, printAdapter,
				new PrintAttributes.Builder().build());
	
		// Save the job object for later status checking
		mPrintJobs.add(printJob);
	}
  ```
  +  自定义文档（内容）打印：
    +  可以先把View画到bitmap中，然后打印bitmap
	+  也可以[实现PrintDocumentAdapter](http://developer.android.com/training/printing/custom-docs.html)，打印自定义内容
  