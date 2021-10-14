---
toc: true
layout: post
title: 學習 Android 背景執行服務 Android Services
description: 學習 Android 背景執行服務 Android Services
categories: [Android, Android Services]
date: 2017-04-05 00:44:58
---

最近為了設計無網路情況下，將資料儲存在SQLite中，而偵測用戶網路連接後，將資料上傳。必須使用到Service的功能，所以剛好有機會把Service學得更深!

Android的`Service`是Android的四大組件之一。不同於`Activity`的生命週期，`Service`可以在背景不斷的工作，直到停止或是系統無法提供資源為止。通常運用在後台播放音樂、定時檢查更新資料，或是執行很久的工作，如上傳及下載檔案。

# 基本概念

要讓Service能在後台運行，必須定義一個繼承Service的類別，並在`AndroidManifest.xml`宣告這個Service以及其他相關屬性，並了解如何啟動他。Service的呼叫分為兩種 :

## startService
當Service使用`startService()`啟動後，就算Activity被關閉，Service也會持續在Background工作著。Service與Acitivty之間沒有什麼交集，Service完成任務也不會回傳東西給原來的應用程式。

## bindService
呼叫`bindService()`則可以讓Activity與Service進行綁定，例如在Activity中指定Service執行某些任務。而bindService也能夠讓Service被多個不同的應用程式呼叫，達到跨應用程式的互動與協作。

![兩種啟動Service方式的生命週期](https://developer.android.com/images/service_lifecycle.png "https://developer.android.com/guide/components/services.html")

1. 應用程式呼叫`startService()`後，系統呼叫Service類別內的`onCreate()`，接著呼叫`onStartCommand()`利用Intent提供的參數做事。直到工作結束或是應用程式呼叫`stopService()`才會停止。

2. 應用程式呼叫`bindService()`與Service綁定前，應用程式需要建立一個`serviceConnection`物件，呼叫後若Service尚未啟動，Service類別內的`onCreate()`就會被呼叫，使用這種方法啟動Service的應用程式，可以透過`onBind()`方法取得IBinder物件，接下來就可以透過IBinder物件來取得Service的事件。

# 做法

## 在AndroidManifest.xml檔中新增定義。

```xml
<? xml version = "1.0" encoding = "utf-8" ?>    
< manifest xmlns:android = "http://schemas.android.com/apk/res/android"   
    package = "com.example.serviceTest"  
    android:versionCode = "1"  
    android:versionName = "1.0" >   
  
    < uses-sdk  
        android:minSdkVersion = "14"  
        android:targetSdkVersion = "17" />
  
    < application  
        android:allowBackup = "true"
        android:icon = "@drawable/ic_launcher"  
        android:label = "@string/app_name"  
        android:theme = "@style/AppTheme" >   
    ……  
        < service android:name = "com.example.serviceTest.MyService">
        </ service >  
    </ application >  
  
</ manifest >  
```

## 實作 Service 類別

實作一個繼承自Service類別的物件。bindService與startService兩者能夠同時存在並不衝突，而且在這次專案裡剛好都會使用到，所以只要同時實作onStartCommand()與onBind()兩個事件即可。

```java
public class MyService extends Service {   
  
    public static final String TAG = "MyService";
  
    private MyBinder mBinder = new MyBinder();
  
    @Override  
    public void onCreate() {   
        super .onCreate();  
        Log.d(TAG, "onCreate() executed");
    }  
  
    @Override  
    public int onStartCommand(Intent intent, int flags, int startId) {   
        Log.d(TAG, "onStartCommand() executed");  
        // 執行任務
        return super.onStartCommand(intent, flags, startId);
    }  
  
    @Override  
    public void onDestroy() {   
        super.onDestroy();  
        Log.d(TAG, "onDestroy() executed");  
    }  
  
    @Override  
    public IBinder onBind(Intent intent) {  
        return mBinder;  
    }  
  
    class MyBinder extends Binder {  
        public void startDownload() {   
            Log.d("TAG", "startDownload() executed");  
            // 執行任務
        }  
    }  
}  
```

## 在 Activity 實作 (1)

在Activity中呼叫startService()來執行第一種啟動Service的方法。

```java
@Override
public View onCreate(Bundle savedInstanceState)
{
    setContentView(R.layout.activity_main);
    super.onCreate(savedInstanceState);
    Intent intent = new Intent(this, MyService.class); 
    this.startService(intent);
}
```

## 在 Activity 實作 (2)

建立 MyBinder 與 ServiceConnection 物件，並呼叫bindService()執行第二種方法。

```java
public class MainActivity extends Activity {

	private MyService.MyBinder myBinder;  
	private ServiceConnection mServiceConnection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName className, IBinder binder) {
            myBinder = (MyService.MyBinder) service;  
            myBinder.startDownload();  
        }

        @Override
        public void onServiceDisconnected(ComponentName className) {
        }
    }

    @Override
    public void onResume() {
        super.onResume();
        Intent intent = new Intent(this, MyService.class);
        this.bindService(intent, mServiceConnection, Context.BIND_AUTO_CREATE);
    }

    @Override
    public void onPause() {
        super.onPause();
        this.unbindService(mServiceConnection);
    }
}
```

以上就是關於Android Services的基礎操作，另外還有很多Service相關的東西，如IntentService、Service和Thread的關係、如何運行前台的Service、或者是遠端Service的協作等著學習。

# Reference
- [Services \| Android Developers](https://developer.android.com/guide/components/services.html?hl=zh-tw)
- [Android Service完全解析，关于服务你所需知道的一切(上)](http://blog.csdn.net/guolin_blog/article/details/11952435)
- [健行科技Android手機程式設計人才培訓班 - Service背景執行程式](http://www.aaronlife.com/teaching/uch_android_2015-02-06_00.html)
- [《Android》『Service』- 背景執行服務的基本用法](http://xnfood.com.tw/android-service/)