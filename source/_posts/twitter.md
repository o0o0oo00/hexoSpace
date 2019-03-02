---
title: twitter 登录 分享
date: 2018-09-18 15:51:54
categories: 技术
tag : android
---
### Twitter 登录
#### 1、添加依赖：
**implementation 'com.twitter.sdk.android:twitter:3.1.1'  
implementation 'com.twitter.sdk.android:tweet-composer:3.1.1'**
<!--more-->
#### 2、添加KEY SECRET
在value string 文件夹下面
```
<string name="com.twitter.sdk.android.CONSUMER_KEY">uy0CK2sjygumtE7J445NQB9s4</string>  
```
```
<string name="com.twitter.sdk.android.CONSUMER_SECRET">3bI33NoTXqTGV5JIU0MmVlllqygY55CHWNOXTsTnKiho3MbbPO</string>
```
#### 3、初始化Twitter
Application 中 **Twitter.initialize(this)**

#### 4、Activity / fragment 的布局中加loggin button
```
<com.twitter.sdk.android.core.identity.TwitterLoginButton
android:id="@+id/loginButton"
android:layout_width="match_parent"
android:layout_height="wrap_content" />
```
##### 登录回调
```
loginButton.callback = object : Callback<TwitterSession>() {
override fun success(result: Result<TwitterSession>) {
Toast.makeText(this@MainActivity, "login success", Toast.LENGTH_SHORT).show()
}

override fun failure(exception: TwitterException) {
Toast.makeText(this@MainActivity, "login failure", Toast.LENGTH_SHORT).show()
}
}

```
##### 启动logginButton回调
```
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
super.onActivityResult(requestCode, resultCode, data)
loginButton.onActivityResult(requestCode, resultCode, data)
}

```
##### 拿到用户信息
```
fun requestUser() {
val apiClient = TwitterCore.getInstance().apiClient
val call = apiClient.accountService.verifyCredentials(true, false, true)
call.enqueue(object : Callback<User>() {
override fun success(result: Result<User>) {
Log.e("123", result.data?.let {
it.name + it.id + it.idStr
})
}

override fun failure(exception: TwitterException?) {
Log.e("123", "get user failure")
}

})
}
```

### Twitter分享

#### 两种方式分享
##### 1、Launching Twitter Composer 无回调
```
try {
val builder = TweetComposer.Builder(this@MainActivity)
.url(URL("https://www.google.com/"))
.text("nice to meet you")
builder.show()
} catch (e: MalformedURLException) {
e.printStackTrace()
}
```
##### 2、Twitter Kit Native Composer 有回调
```
val session = TwitterCore.getInstance().sessionManager?.activeSession
session?.let {
val intent = ComposerActivity.Builder(this@MainActivity)
.session(session)
.text("Love where you work")
.hashtags("#twitter")
.createIntent()
startActivity(intent)
}
```
回调采用的是广播方式：
```
class MyResultReceiver : BroadcastReceiver() {
override fun onReceive(context: Context?, intent: Intent) {
if (TweetUploadService.UPLOAD_SUCCESS == intent.action) {
// success Twitter分享成功的回调
Toast.makeText(context, "分享成功的回调", Toast.LENGTH_SHORT).show()
val tweetId = intent.extras.getLong(TweetUploadService.EXTRA_TWEET_ID);
} else {
Toast.makeText(context, "分享failure的回调", Toast.LENGTH_SHORT).show()
// failure
//            val retryIntent = intent.extras.getParcelable(TweetUploadService.EXTRA_RETRY_INTENT);
}
}
}
```
注册广播：
```
<receiver
android:name=".MyResultReceiver"
android:exported="false">
<intent-filter>
<action android:name="com.twitter.sdk.android.tweetcomposer.UPLOAD_SUCCESS" />
<action android:name="com.twitter.sdk.android.tweetcomposer.UPLOAD_FAILURE" />
<action android:name="com.twitter.sdk.android.tweetcomposer.TWEET_COMPOSE_CANCEL"/>
</intent-filter>
</receiver>
```
