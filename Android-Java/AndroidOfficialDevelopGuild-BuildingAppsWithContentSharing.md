#Building Apps with Content Sharing
+  Sharing Simple Data
  +  Intent && ActionProvider
  +  发送数据（发起intent调起其他app处理）
    +  Send Text Content
	+  Send Binary Content
	+  Send Multiple Pieces of Content
  +  接收数据
    +  AndroidManifest.xml中为Activity定义`<intent-filter>`
	+  在Activity的onCreate中调用`getIntent()`获取action、数据，并进行处理
+  Sharing Files
  +  唯一“安全”的方式就是：将文件对应的URI通过Intent发送出去，并为该URI提供临时的访问权限。而这些步骤都可以通过`FileProvider`完成
  +  在AndroidManifest.xml中声明provider
	```xml
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
		package="com.example.myapp">
		<application
			...>
			<provider
				android:name="android.support.v4.content.FileProvider"
				android:authorities="com.example.myapp.fileprovider"
				android:grantUriPermissions="true"
				android:exported="false">
				<meta-data
					android:name="android.support.FILE_PROVIDER_PATHS"
					android:resource="@xml/filepaths" />
			</provider>
			...
		</application>
	</manifest>
	```
  +  `<meta-data>`指定描述要分享的目录的xml文件
  +  指定要分享的目录
	```xml
	<paths>
		<files-path path="images/" name="myimages" />
	</paths>
	```
  +  `<paths>`标签可以有多个子标签，`<files-path>`指定app的files目录下的分享目录名，`<external-path>`指定外部存储的分享目录名，`<cache-path>`指定app的cache目录下的分享目录名；分享路径只能在xml中描述；
  +  如上配置后，需要访问files/images/default_image.jpg时，对应uri为：`content://com.example.myapp.fileprovider/myimages/default_image.jpg`
  +  Receive File Requests
    +  定义一个Selection Activity，响应Intent action，例如：`ACTION_PICK`
		```xml
		<activity
			android:name=".FileSelectActivity"
			android:label="@"File Selector" >
			<intent-filter>
				<action
					android:name="android.intent.action.PICK"/>
				<category
					android:name="android.intent.category.DEFAULT"/>
				<category
					android:name="android.intent.category.OPENABLE"/>
				<data android:mimeType="text/plain"/>
				<data android:mimeType="image/*"/>
			</intent-filter>
		</activity>
		```
	+  其他app发起该intent，通过`startActivityForResult()`和`onActivityResult()`中发起请求、处理结果
	+  在Selection Activity的onCreate函数中，解析其他app的请求
		```java
		File requestFile = new File(mImageFilename[position]);
		// Use the FileProvider to get a content URI
		try {
			fileUri = FileProvider.getUriForFile(
					MainActivity.this,
					"com.example.myapp.fileprovider",
					requestFile);
		} catch (IllegalArgumentException e) {
			Log.e("File Selector",
					"The selected file can't be shared: " +
					clickedFilename);
		}
		```
	+  赋予临时访问权限
		```java
		mResultIntent = new Intent("com.example.myapp.ACTION_RETURN_FILE");
		if (fileUri != null) {
			// Grant temporary read permission to the content URI
			mResultIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
		}
		```
	+  使用`setFlags()`赋予临时访问权限更安全，`Context.grantUriPermission()`赋予的权限只有手动调用`Context.revokeUriPermission()`才会被移除
	+  返回结果
		```java
		mResultIntent.setDataAndType(
				fileUri,
				getContentResolver().getType(fileUri));
		// Set the result
		MainActivity.this.setResult(Activity.RESULT_OK,
				mResultIntent);
		finish();
		```
  +  Requesting a Shared File
    +  通常流程是：app发起一个带有请求的intent，分享文件的app的相应Activity被启动，该Activity显示文件列表，用户选择文件后返回被选中的文件的Uri
	+  发起请求
		```java
		mRequestFileIntent = new Intent(Intent.ACTION_PICK);
		mRequestFileIntent.setType("image/jpg");
		startActivityForResult(mRequestFileIntent, 0);
		```
	+  访问返回的文件
		```java
		@Override
		public void onActivityResult(int requestCode, int resultCode,
				Intent returnIntent) {
			// If the selection didn't work
			if (resultCode != RESULT_OK) {
				// Exit without doing anything else
				return;
			} else {
				// Get the file's content URI from the incoming Intent
				Uri returnUri = returnIntent.getData();
				/*
				* Try to open the file for "read" access using the
				* returned URI. If the file isn't found, write to the
				* error log and return.
				*/
				try {
					/*
					* Get the content resolver instance for this context, and use it
					* to get a ParcelFileDescriptor for the file.
					*/
					mInputPFD = getContentResolver().openFileDescriptor(returnUri, "r");
				} catch (FileNotFoundException e) {
					e.printStackTrace();
					Log.e("MainActivity", "File not found.");
					return;
				}
				// Get a regular file descriptor for the file
				FileDescriptor fd = mInputPFD.getFileDescriptor();
				...
			}
		}
		```
  +  Retrieving File Information
    +  MIME Type
	```java
	Uri returnUri = returnIntent.getData();
    String mimeType = getContentResolver().getType(returnUri);
	```
	+  File's Name and Size
	```java
    Uri returnUri = returnIntent.getData();
    Cursor returnCursor =
            getContentResolver().query(returnUri, null, null, null, null);
	int nameIndex = returnCursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
    int sizeIndex = returnCursor.getColumnIndex(OpenableColumns.SIZE);
	nameView.setText(returnCursor.getString(nameIndex));
    sizeView.setText(Long.toString(returnCursor.getLong(sizeIndex)));
	```
+  Sharing Files with NFC
  +  Android Beam，大文件传输，from 4.1 API 16
  +  Android Beam NDEF，小数据传输，from 4.0 API 14
  +  发送文件
    +  权限：`<uses-permission android:name="android.permission.NFC" />`，`<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />`
	+  NFC feature：`<uses-feature android:name="android.hardware.nfc" android:required="true" />`
	+  minSdkVersion >= 16
	+  检查是否支持
		```java
		if (PackageManager.hasSystemFeature(PackageManager.FEATURE_NFC) &&
				Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
			mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
		}
		```
	+  创建提供文件的回调
		```java
		// List of URIs to provide to Android Beam
		private Uri[] mFileUris = new Uri[10];
		...
		/**
		* Callback that Android Beam file transfer calls to get
		* files to share
		*/
		private class FileUriCallback implements
				NfcAdapter.CreateBeamUrisCallback {
			public FileUriCallback() {
			}
			/**
			* Create content URIs as needed to share with another device
			*/
			@Override
			public Uri[] createBeamUris(NfcEvent event) {
				return mFileUris;
			}
		}

		...
        // Android Beam file transfer is available, continue
        ...
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        /*
         * Instantiate a new FileUriCallback to handle requests for
         * URIs
         */
        mFileUriCallback = new FileUriCallback();
        // Set the dynamic callback for URI requests.
        mNfcAdapter.setBeamPushUrisCallback(mFileUriCallback,this);
        ...
		```
	+  设置需要发送的文件
		```java
		/*
         * Create a list of URIs, get a File,
         * and set its permissions
         */
        private Uri[] mFileUris = new Uri[10];
        String transferFile = "transferimage.jpg";
        File extDir = getExternalFilesDir(null);
        File requestFile = new File(extDir, transferFile);
        requestFile.setReadable(true, false);
        // Get a URI for the File and add it to the list of URIs
        fileUri = Uri.fromFile(requestFile);
        if (fileUri != null) {
            mFileUris[0] = fileUri;
        } else {
            Log.e("My Activity", "No File URI available for file.");
        }
		```
  +  接收文件
    +  设置intent-filter
		```xml
		<activity
			android:name="com.example.android.nfctransfer.ViewActivity"
			android:label="Android Beam Viewer" >
			...
			<intent-filter>
				<action android:name="android.intent.action.VIEW"/>
				<category android:name="android.intent.category.DEFAULT"/>
				...
			</intent-filter>
		</activity>
		```
	+  权限：`<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />`
	+  获取接收文件的路径
		```java
        // Get the Intent action
        mIntent = getIntent();
        String action = mIntent.getAction();
        /*
         * For ACTION_VIEW, the Activity is being asked to display data.
         * Get the URI.
         */
        if (TextUtils.equals(action, Intent.ACTION_VIEW)) {
            // Get the URI from the Intent
            Uri beamUri = mIntent.getData();
            /*
             * Test for the type of URI, by getting its scheme value
             */
            if (TextUtils.equals(beamUri.getScheme(), "file")) {
                mParentPath = handleFileUri(beamUri);
            } else if (TextUtils.equals(
                    beamUri.getScheme(), "content")) {
                mParentPath = handleContentUri(beamUri);
            }
        }
		```
	+  读取
		```java
		String fileName = beamUri.getPath();
		File copiedFile = new File(fileName);
		```
	+  根据不同的content provider读取文件
		```java
		...
		public String handleContentUri(Uri beamUri) {
			// Position of the filename in the query Cursor
			int filenameIndex;
			// File object for the filename
			File copiedFile;
			// The filename stored in MediaStore
			String fileName;
			// Test the authority of the URI
			if (!TextUtils.equals(beamUri.getAuthority(), MediaStore.AUTHORITY)) {
				/*
				* Handle content URIs for other content providers
				*/
			// For a MediaStore content URI
			} else {
				// Get the column that contains the file name
				String[] projection = { MediaStore.MediaColumns.DATA };
				Cursor pathCursor =
						getContentResolver().query(beamUri, projection,
						null, null, null);
				// Check for a valid cursor
				if (pathCursor != null &&
						pathCursor.moveToFirst()) {
					// Get the column index in the Cursor
					filenameIndex = pathCursor.getColumnIndex(
							MediaStore.MediaColumns.DATA);
					// Get the full file name including path
					fileName = pathCursor.getString(filenameIndex);
					// Create a File object for the filename
					copiedFile = new File(fileName);
					// Return the parent directory of the file
					return new File(copiedFile.getParent());
				} else {
					// The query didn't work; return null
					return null;
				}
			}
		}
		...
		```
