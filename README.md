# SocialSDKAndroid
对第三方社会化sdk的集成和二次封装，比如第三方授权登录、第三方分享等

> 欢迎发issue建议新的功能点和渠道集成

原文阅读:

[http://www.jianshu.com/p/4ec1d9c15763](http://www.jianshu.com/p/4ec1d9c15763)


## 0 版本更新

|版本号|更新内容|
|---|---|
|1.0|集成微信授权登录,5种分享媒介,微信会话分享,微信朋友圈分享|
|1.1|集成QQ授权登录,QQ分享,QQ空间分享|
|1.2|将微信appsecret移除,客户端不放appsecret|
|1.3|增加新浪微博授权登录和分享|
|1.4|增加可选的Api调用|

## 1 目录介绍

采用了jar包的方式封装sdk,需要使用时可以引入social_sdk.jar再搭配需要的平台sdk使用.
这种方式可以减少sdk的体积,需要什么平台就引入哪个平台.更为合理.

- social_sdk/ sdk的开发源码module 开发完成后用gradle中makejar打成jar包
- social_sdk.jar sdk的jar包 直接使用.搭配所需的平台sdk包.
- weixin_sdk/ 微信sdk
- qq_sdk/ qq sdk
- sina_weibo_sdk/ 新浪微博 sdk
- SampleCode/ 一个示例代码(非可运行项目)

## 2 功能介绍

### 2.1 授权登录

1. 微信授权登录
2. QQ授权登录
3. 新浪微博授权登录

### 2.2 分享

#### 2.2.1 分享媒介

1. 文字
1. 图片
1. 音乐
1. 视频
1. 网页

#### 2.2.2 分享平台

1. 微信会话分享
1. 微信朋友圈分享
1. QQ分享
1. QQ空间分享
1. 新浪微博分享

### 2.3 API调用

当第三方授权成功后获取到access_token，会提供一些API查询用户信息，刷新access_token等。现在逐步开始封装API，方便使用。

#### 2.3.1 微信API (WXApi)

1. getAccessToken(String wxAppId, String wxAppSecret,
                                      String code,
                                      final Callback callback)

   ```java
	/**
	 * 获取access_token
	 * @param wxAppId wx appid
	 * @param wxAppSecret wx appsecret
	 * @param code 调用微信登录获取的code
	 * @param callback
	 */
   ```
   
1. getUserInfo(String openid, String access_token,
                                   final Callback callback)
                                   
   ```java
   /**
     * 获取用户信息
     * @param openid openid
     * @param access_token access_token
     * @param callback
     */
   ```

## 3 开发说明

### 3.1 准备

将social_sdk.jar和需要的平台sdk放入项目中引用.

AndroidManifest加上以下基本的权限(之后各个平台会注册一些不同的信息后面会说明)

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
```

### 3.2 配置平台信息

在项目入口(或者在调用前)需要配置平台的信息,配置一次即可.

```java
PlatformConfig.setWeixin(WX_APPID);
PlatformConfig.setQQ(QQ_APPID);
PlatformConfig.setSinaWB(SINA_WB_APPKEY);
```

### 3.3 接口使用说明

调用方式使用api调用登录或者分享接口,在参数中区别平台,实现回调接收成功 取消或者失败的结果.

示例如下:(某些平台会有一些特殊处理后面会在平台中说明)

初始化api:

```java
SocialApi mSocialApi = SocialApi.get(getApplicationContext());
```

登录授权:
```java
mSocialApi.doOauthVerify(this, PlatformType.WEIXIN, new AuthListener() {
    @Override
    public void onComplete(PlatformType platform_type, Map<String, String> map) {
        Log.i("tsy", "oncomplete:" + map);
    }

    @Override
    public void onError(PlatformType platform_type, String err_msg) {
        Log.i("tsy", "onError:" + err_msg);
    }

    @Override
    public void onCancel(PlatformType platform_type) {
        Log.i("tsy", "onCancel");
    }
});
```

分享:
```java

//分享媒介 后面有详细介绍
ShareWebMedia shareMedia = new ShareWebMedia();
shareMedia.setTitle("分享网页测试");
shareMedia.setDescription("分享网页测试");
shareMedia.setWebPageUrl("http://www.baidu.com");
shareMedia.setThumb(BitmapUtils.readBitMap(getApplicationContext(), R.mipmap.ic_launcher));

mSocialApi.doShare(this, PlatformType.WEIXIN, shareMedia, new ShareListener() {
    @Override
    public void onComplete(PlatformType platform_type) {
        Log.i("tsy", "share onComplete");
    }

    @Override
    public void onError(PlatformType platform_type, String err_msg) {
        Log.i("tsy", "share onError:" + err_msg);
    }

    @Override
    public void onCancel(PlatformType platform_type) {
        Log.i("tsy", "share onCancel");
    }
});

```

### 3.4 分享媒介

现在集成了文字分享,图片分享,音乐分享,视频分享,网页分享5种分享媒介.不同的平台可能只有其中某几种.

#### 3.4.1 文字分享

```java
ShareTextMedia shareMedia = new ShareTextMedia();
shareMedia.setText("分享文字测试");
```
#### 3.4.2 图片分享

```java
ShareImageMedia shareMedia = new ShareImageMedia();
shareMedia.setImage(BitmapUtils.readBitMap(getApplicationContext(), R.mipmap.ic_launcher));
```

#### 3.4.3 音乐分享

```java
ShareMusicMedia shareMedia = new ShareMusicMedia();
shareMedia.setTitle("分享音乐测试");
shareMedia.setDescription("分享音乐测试");
shareMedia.setMusicUrl("http://idg-tangsiyuan.tunnel.nibaguai.com/splash/music.mp3");
shareMedia.setThumb(BitmapUtils.readBitMap(getApplicationContext(), R.mipmap.ic_launcher));
```

#### 3.4.4 视频分享

```java
ShareVideoMedia shareMedia = new ShareVideoMedia();
shareMedia.setTitle("分享视频测试");
shareMedia.setDescription("分享视频测试");
shareMedia.setVideoUrl("http://idg-tangsiyuan.tunnel.nibaguai.com/splash/music.mp3");
shareMedia.setThumb(BitmapUtils.readBitMap(getApplicationContext(), R.mipmap.ic_launcher));
```

#### 3.4.5 网页分享

```java
ShareWebMedia shareMedia = new ShareWebMedia();
shareMedia.setTitle("分享网页测试");
shareMedia.setDescription("分享网页测试");
shareMedia.setWebPageUrl("http://www.baidu.com");
shareMedia.setThumb(BitmapUtils.readBitMap(getApplicationContext(), R.mipmap.ic_launcher));
```

## 4 第三方平台接入

### 4.1 微信

#### 4.1.1 集成sdk

将目录中的weixin_sdk_v3.1.1.jar放入项目.

#### 4.1.2 配置

创建固定activity: 包名.wxapi.WXEntryActivity.java
该activity继承WXCallbackActivity类.

```java
...
import com.tsy.sdk.social.weixin.WXCallbackActivity;

/**
 * Created by tsy on 16/8/4.
 */
public class WXEntryActivity extends WXCallbackActivity {

}
```

AndroidManifest中添加:

```java
<activity
    android:name=".wxapi.WXEntryActivity"
    android:configChanges="keyboardHidden|orientation|screenSize"
    android:exported="true"
    android:screenOrientation="portrait"
    android:theme="@android:style/Theme.Translucent.NoTitleBar" />
```

#### 4.1.3 常量定义

设置配置信息:

```java
PlatformConfig.setWeixin(WX_APPID);
```

PlatformType:

微信:PlatformType.WEIXIN(可用于登录和微信回话分享)

朋友圈:PlatformType.WEIXIN_CIRCLE(用于微信朋友圈分享)

#### 4.1.4 注意

使用微信登录分享需要签名打包，并且签名和包名要和微信平台填入的信息一致。


### 4.2 QQ

#### 4.2.1 集成sdk

将目录中的qq_mta-sdk-1.6.2.jar和qq_sdk_v3.1.0.jar放入项目.

#### 4.2.2 配置

AndroidManifest中添加:

```java
<!--qq-->
<activity
    android:name="com.tencent.tauth.AuthActivity"
    android:noHistory="true"
    android:launchMode="singleTask" >
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="tencent你的appid" />
    </intent-filter>
</activity>
<activity
    android:name="com.tencent.connect.common.AssistActivity"
    android:screenOrientation="portrait"
    android:theme="@android:style/Theme.Translucent.NoTitleBar" />
```

#### 4.2.3 常量定义

设置配置信息:

```java
PlatformConfig.setQQ(QQ_APPID);
```

PlatformType:

微信:PlatformType.QQ(可用于登录和QQ分享)

朋友圈:PlatformType.QZONE(用于qq控件分享)

#### 4.2.4 注意

使用QQ登录需要签名打包，并且签名和包名要和QQ开放平台填入的信息一致。

### 4.3 新浪微博

#### 4.3.1 集成sdk

将目录中的weiboSDKCore_3.1.4.jar放入项目.

将所有so文件统一放到项目的目录app/jniLibs中(和libs同级), 然后在gradle中加上

```
android {

    ...

    //引入微博的所有so库
    sourceSets {
        main {
            jniLibs.srcDirs = ['jniLibs']
        }
    }
}
```

#### 4.3.2 配置

AndroidManifest中添加:

```java
<activity
    android:name="com.sina.weibo.sdk.component.WeiboSdkBrowser"
    android:configChanges="keyboardHidden|orientation"
    android:exported="false"
    android:windowSoftInputMode="adjustResize"></activity>
```

并且在发起分享的activity页面的AndroidManifest页面中加上

```java
<activity android:name="com.tsy.girl.MainActivity">     <!--发起分享的页面-->
    <intent-filter>
        <action android:name="com.sina.weibo.sdk.action.ACTION_SDK_REQ_ACTIVITY" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

然后在发起分享的Activity中添加以下代码.(假如是MainActivity)

实现IWeiboHandler.Response接口, 然后在实现的方法中写:

```java
@Override
public void onResponse(BaseResponse baseResponse) {
    ((SinaWBHandler)mSocialApi.getSSOHandler(PlatformType.SINA_WB)).onResponse(baseResponse);
}
```

实现onNewIntent和onActivityResult方法:

```java
@Override
protected void onNewIntent(Intent intent) {
    ((SinaWBHandler)mSocialApi.getSSOHandler(PlatformType.SINA_WB)).onNewIntent(intent, this);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    mSocialApi.onActivityResult(requestCode, resultCode, data);
}

```

在onCreate中添加以下:

```java
if (savedInstanceState != null) {
    ((SinaWBHandler)mSocialApi.getSSOHandler(PlatformType.SINA_WB)).onNewIntent(getIntent(), this);
}
```

然后正常发起授权或者分享代码即可.

#### 4.3.3 常量定义

设置配置信息:

```java
PlatformConfig.setSinaWB(SINA_WB_APPKEY);
```

#### 4.3.4 注意

使用新浪登录分享需要签名打包，并且签名和包名要和新浪平台填入的信息一致。

## 欢迎关注我的公众号

![我的公众号](https://github.com/tsy12321/PayAndroid/blob/master/wxmp_avatar.jpg)
