# Building Apps with Contacts & Sign-In

## Accessing Contacts Data
+  官方文档上使用[`CursorLoader`](http://developer.android.com/reference/android/support/v4/content/CursorLoader.html)加载本地通讯录列表，然后用`SimpleCursorAdapter`显示在`ListView`中，示例比较繁琐
+  项目实践中的代码：

```java
Cursor cur = contentResolver.query(ContactsContract.CommonDataKinds.Phone.CONTENT_URI, 
      null, null, null, null);
if (cur == null || cur.getCount() == 0) {
    if (cur != null) {
        cur.close();
    }
    throw new RuntimeException("Permission denied"); // 不同系统行为不一样，有的是cursor为null，有的是cursor数据为空
}
while (cur.moveToNext()) {
    String name = cur.getString(
            cur.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
    String phone = cur.getString(
            cur.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
    //...
}
cur.close();
```

+  可以通过发送`Intent`实现对通讯录的修改，会启动系统通讯录应用，在其中让用户进行确认修改，这种方式无需修改通讯录权限：

```java
Intent intent = new Intent(Intents.Insert.ACTION);
intent.setType(ContactsContract.RawContacts.CONTENT_TYPE);
intent.putExtra(Intents.Insert.EMAIL, mEmailAddress)
      .putExtra(Intents.Insert.EMAIL_TYPE, CommonDataKinds.Email.TYPE_WORK)
      .putExtra(Intents.Insert.PHONE, mPhoneNumber)
      .putExtra(Intents.Insert.PHONE_TYPE, Phone.TYPE_WORK);
startActivity(intent);
```

+  API 14以上的系统，启动通讯录应用修改完通讯录后，按下返回键不会回到原来的APP，API 15以上可以用以下办法解决：

```java
// Sets the special extended data for navigation
editIntent.putExtra("finishActivityOnSaveCompleted", true);
```

## [Adding Sign-In](http://developer.android.com/intl/zh-cn/training/sign-in/index.html)
Google提供的授权API，让用户可以使用Google账户授权登录app，享用Google账户关联的优势。
