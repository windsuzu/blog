---
toc: true
layout: post
title: 學習 RxJava 結合 Retrofit
description: 學習 RxJava 結合 Retrofit
categories: [Android, RxJava, Retrofit]
date: 2017-05-05 18:14:45
---

個別學習完 RxJava 與 Retrofit 之後，是令人期待的合體了呢 ! 查看前面的文章: 

- [學習 Android RxJava]({% post_url 2017-05-05-learn-android-rxjava %})
- [學習 Android Retrofit 2]({% post_url 2017-05-02-learn-android-retrofit2 %})

# Setup

設置 Gradle 除了 RxJava 與 Retrofit 的 Library 以外，還要再多設置讓兩者能夠結合的 adapter 套件。

```gradle
compile "io.reactivex.rxjava2:rxjava:2.x.y"
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'

compile 'com.squareup.retrofit2:retrofit:2.2.0'
compile 'com.squareup.retrofit2:converter-gson:2.2.0'
compile 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
```

{% include alert.html text="要記得 retrofit 與他的快樂夥伴的版本要一致" %}

在設置這邊發生了好多錯誤，原來之前學習時用的是 RxJava1，所以跟 Retrofit2 的 Adapter 配不起來； 1 和 2 有一些差別，可以到 [RxJava Github - What's different in 2.0](https://github.com/ReactiveX/RxJava/wiki/What%27s-different-in-2.0) 查看。


# 實作 Retrofit Service Interface

一樣拿 GitHub Api 測試看看 :

```java
public interface GitHubService {
    public final static String BASEURL = "http://api.github.com/";

    @GET("users/{userId}")
    Observable<GitHubUser> getUser(@Path("userId") String userId);
}
```

這邊要注意的是原本的 Call<GitHubUser> 變為 Observable<GitHubUser> ，因為是用 RxJava 觀察者模式訂閱，所以必須要返回一個 Observable 對象，才能夠進行接續的 `subscribeOn()` 、 `subScribe()` 等 RxJava 操作。其餘的都跟只有 Retrofit 時一樣。


# 創建 Retrofit 元件

```java
public Retrofit provideRetrofit(String baseUrl) {
    return new Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create())
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .baseUrl(baseUrl)
            .build();
}
```

這邊除了原本添加的 Gson 轉換器，還要新增 Retrofit 與 RxJava 相依的 RxJava2CallAdapterFactory 。

# 實作 Retrofit + RxJava 異步網路請求

```java
GitHubService service = provideRetrofit(GitHubService.BASEURL).create(GitHubService.class);

service.getUser(USERID)
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(user -> Log.d(TAG, user.getName()));
```

先利用 `getUser()` 獲得 Observable ，再開始使用 RxJava 的方法，能夠歷經 map 、 fliter 、 flatMap ，最後傳給 Observer 進行主執行緒的操作。

# 結語
除此之外，若是像設計登入流程，伺服器可能會先回傳一個 token ，需要再以 token 去取得用戶資料。這時可能就要使用 flatMap 再一次發送 token 取得資料。

或是登入時需要實作 `onError()` 的響應，一方面又想要維持 lambda 語法的簡潔，那就必須要另外新增兩個 `Action (Consumer)` ，來實作 `onNext()` 、 `onError()` 的方法，再放入 `subScribe` 中。

> 上面說的登入設計可以參考這篇文章: 
> - [【Android】RxJava + Retrofit完成网络请求](http://www.jianshu.com/p/1fb294ec7e3b)