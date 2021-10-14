---
toc: true
layout: post
title: 學習 Android RxJava
description: 學習 Android RxJava
categories: [Android, RxJava]
date: 2017-05-05 16:14:07
---

# 為什麼要學習 RxJava 

終於寫到了 RxJava ，前陣子在 [掘金](https://juejin.im) 不斷看到這個名字，一直想把他學起來應用在開發上。 看了好多文章，也實際操作，延伸學習了不少新的東西。像是更了解 Android 的執行緒、觀察者模式、 Java 8 的 lambda 語法。雖然我這個新手還沒有做過很大的 Project ，但相信 RxJava 會成為未來一個很棒的工具。

記得什麼基礎都沒有時，為了要把網站的圖下載下來顯示到 imageView 中，使用了同步請求，把主執行緒塞爆 UI 當的要死。接著學習了 okhttp、Volley 這些 HTTP 請求的框架，知道了怎麼樣異步處理網路的請求。 但越寫越多，發現自己的程式碼亂七八糟。

RxJava 就是要讓 "異步操作" 更乾淨、簡潔的實作出來，而且比起 Android 內建的 AsyncTask 或 Handler 更能夠應付越寫越複雜的程式碼。搭配上 lambda 語法，看起來也變得很漂亮。

# 原理
RxJava 的概念即是觀察者模式 (Observer Pattern) ，由觀察者 (Observer) 與被觀察者 (Subject) 組成。

<img src="{{site.baseurl}}/images/markdown/learn-android-rxjava/flow1.jpg" width=200>

**觀察者** 透過訂閱 (Subscribe) 或稱註冊 (Register) 的方式，監控著 **被觀察者** 。只要 **被觀察者** 一發生什麼變動，就馬上通知 **觀察者**

就像 Button 註冊 OnClickListener 一樣 :

<img src="{{site.baseurl}}/images/markdown/learn-android-rxjava/flow2.jpg" width=200>

RxJava 中的觀察者為 Observer ， 被觀察者為 Observable ， 訂閱的方法為 subscribe()，流程也是類似方式
<img src="{{site.baseurl}}/images/markdown/learn-android-rxjava/flow3.jpg" width=200>


這邊要注意的是，這些方法在 java 中寫起來是長這樣的 :
``` java
myButton.setOnClickListener(myOnClickListener);
observable.subscribe(observer);
```
咦? 這樣看起來不就是 observable 去訂閱 observer 嗎 ? 我也很納悶，在一篇文章這樣解釋著。

>
> 有人可能會注意到， subscribe() 這個方法有點怪：它看起來是『observable 訂閱了 observer / subscriber』而不是『observer / subscriber 訂閱了 observalbe』，這看起來就像『雜志訂閱了讀者』一樣顛倒了對像關系。這讓人讀起來有點別扭，不過如果把 API 設計成 observer.subscribe(observable) / subscriber.subscribe(observable) ，雖然更加符合思維邏輯，但對流式 API 的設計就造成影響了，比較起來明顯是得不償失的。
> 
> [扔物线 - 給 Android 開發者的 RxJava 詳解](https://gank.io/post/560e15be2dca930e00da1083)

---

# Hello, World !
先從最基本的回傳字串實際操作看看 !

## 設置 Gradle
``` gradle
    compile 'io.reactivex:rxjava:1.0.14'
    compile 'io.reactivex:rxandroid:1.0.1'
```

## 觀察者 Observer
先從創建一個觀察者 Observer 看看 :
``` java
Observer<String> observer = new Observer<String>() {
    @Override
    public void onNext(String s) {
        Log.d(TAG, "Item: " + s);
    }

    @Override
    public void onCompleted() {
        Log.d(TAG, "Completed!");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, e.getMessage());
    }
};
```
RxJava 的 Observable 發生改變時，會回傳給 Observer 三種方法，分別是 `OnNext()` 、 `OnComplete()` 、 `onError()` :

* OnNext() : Observable 發送過來的訊息，會批次的在這個方法中實現，就像 OnClick() 一樣。
* OnComplete() : 當全部 OnNext() 都被執行完畢，會觸發這個方法。
* onError() : 如果在執行的途中發生錯誤，會馬上執行這個方法，並且中止。


## 被觀察者 Observable
接著創建被觀察者 Observable 
``` java
Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("Hello!");
        subscriber.onNext("World!");
        subscriber.onCompleted();
    }
});
```
RxJava 使用 create() 創建一個 Observable，並且丟入參數 OnSubscribe 儲存回 Observable 中，當 Observable 被訂閱後，OnSubscribe 的 call() 方法就會被呼叫。

那如何訂閱呢 ? 前面有說過 :
``` java
observable.subscribe(observer);
// LogCat 就會依序印出 "Hello!", "World!", "Completed!" 囉
```

---

# 簡潔化
看完了最基礎的用法後，如果每次都要創建 Observer 跟 Observable ，再來 subscribe() 連結關係，好麻煩 !
所以 RxJava 還有好多必學的用法。

## just
使用 Observable.just() 來創建 **被觀察者** ，與 Observable.create() 所產生的結果是一模一樣的。
``` java
Observable observable = Observable.just("Hello!", "World!");
// 一樣會依序印出 "Hello!", "World!", "Completed!"
```

## from
Observable.from() 則是能夠讀取繼承 `Iterable` 的所有類別如 `List`，並且拆開依序發送出來。
``` java
String[] list = {"Hello!", "World!"};
Observable observable = Observable.from(list);
// 一樣會依序印出 "Hello!", "World!", "Completed!"
```


## Action
簡化了 Observable 的方法後，再來是 Observer 這邊了。 subscribe(Observer) 需要回傳三種方法，在示範中，除了 `onNext()` 外，其實我們不怎麼需要回傳 `onComplete()` 與 `onError()` 。

所以 RxJava 提供了 Action 這個 Interface ，包含了 `Action0` 與 `Action1` 。

`Action0` 與 `onComplete` 一樣是沒有參數與返回值的，因此可以利用 `Action0` 實作一個與 `onComplete` 一樣的方法。
``` java
Action0 onCompletedAction = new Action0() {
    @Override
    public void call() {
        Log.d(TAG, "Completed!");
    }
};
```

而需要參數的 onError(err) 與 onNext(obj) ， 則利用相同需要傳入參數與返回值的 Action1 來實作。
```java
Action1<String> onNextAction = new Action1<String>() {
    @Override
    public void call(String s) {
        Log.d(TAG, "Item: " + s);
    }
};
Action1<Throwable> onErrorAction = new Action1<Throwable>() {
    @Override
    public void call(Throwable e) {
        Log.d(TAG, e.getMessage());
    }
};
```

再來就可以依照需要的 Action 傳入 subscribe() 當中 !
``` java
// 只需要定義 onNext()
observable.subscribe(onNextAction); 

// 以此類推
observable.subscribe(onNextAction, onErrorAction);
observable.subscribe(onNextAction, onErrorAction, onCompletedAction);
```

練習總結這三種用法 :
``` java
public void createActions() {
        Observable.just("Hello!", "World!").subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                Log.d(TAG, "Item: " + s);
            }
        });
    }
```
哇嗚，變得簡潔好多 ! 如果再用之後的 lambda 語法將變得無比華麗呢。

---

# 變換函數
什麼 ? 你說到現在只是用很炫的方式把字串 print 出來而已 ? 其實現在才要開始看到 RxJava 真正厲害的地方啦 ! RxJava 提供 map 、 flatmap 、 reduce 等方法，將傳進來的每個對象或對象本身進行加工處理並且轉換。

## map
如果我們有一用戶列表，要抓出全部用戶名字，並添加到新的 List<String> 當中，用一般做法，可能需要用到迴圈之類的麻煩事，但 `map()` 不需要 :
``` java
User user1 = new User();
...

Observable.just(user1, user2, user2)

                // 參數1 : 傳進來的類別
                // 參數2 : 轉換完輸出的類別

                // 讀進每個 User ， 並轉換為 name 這個 String 傳到下一個 subscribe 方法
                .map(new Func1<User, String>() {
                    @Override
                    public String call(User user) {
                        String name = user.getName();
                        return name;
                    }
                })
                // 將讀取的 name 加到新的 List 中
                .subscribe(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        nameList.add(s);
                    }
                });
```
例子中的 `Func1` 這個 Interface ，跟 `Action1` 相似，同樣傳入參數進入的一個方法。不同的是， `Func1` 的 `call()` 是有返回值的 ! 
阿 ! 原來數字是多少就是有幾個返回值呢 ! `FuncX` 就有 X 個返回值 ! 懂了 !


## flatmap
如果這時要把每個用戶的每個文章標題都列出來，用 `map()` 的做法會需要用到迴圈 :
``` java
Observable.just(user1, user2, user2)
                .map(new Func1<User, List<Article>>() {
                    @Override
                    public List<Article> call(User user) {
                        return user.getArticleList();
                    }
                })
                // 將轉換成 List<Article> 傳入 Observer 中
                .subscribe(new Action1<List<Article>>() {
                    @Override
                    public void call(List<Article> articleList) {
                        for (Article a : articleList) {
                        	Log.d(TAG, a.getTitle());
                        }
                    }
                });
```

由於 `map()` 只能夠實現 1對1 的轉換，所以出現另人煩躁的迴圈了。那要怎麼辦才能在 Observer 中每次都傳入一個 Article 類別呢 ? RxJava 還提供了一個 `flatmap()` 的方法。
``` java
Observable.just(user1, user2, user2)

                // User 類別將在 flatmap 中轉換成 Observable<Article> 類別
                .flatmap(new Func1<User, Observable<Article>>() {
                    @Override
                    public Observable<Article> call(User user) {
                        return Observable.from(user.getArticleList());
                    }
                })
                // 即可獲得序列中每個 Article
                .subscribe(new Action1<Article>() {
                    @Override
                    public void call(Article article) {
                        Log.d(TAG, article.getTitle());
                    }
                });
```

<img src="{{site.baseurl}}/images/markdown/learn-android-rxjava/flatmap.jpg" width=400>

| 角色 | 執行                                                                            |
| ---- | ------------------------------------------------------------------------------- |
| 紅色 | 進到 flatmap 後，經過新創建的 Observable 將 Article 1 和 Article 2 交給 Observer |
| 紫色 | 進到 flatmap 後，經過新創建的 Observable 將 Article 3 和 Article 4 交給 Observer |

## 其他轉換函數

- filter : 過濾掉無意義的數據
- reduce : 跟 flatmap 相反，把數個數組組合成一個數據

更多請參考 : [谁来讲讲Rxjava、rxandroid中的操作符的作用?](https://www.zhihu.com/question/32209660)

---

# Scheduler
另外一個 RxJava 強大的原因，就是能夠隨意的切換執行緒 (Thread)。我們知道，如果要從事一些網路下載、加載圖片等較為耗時的工作，不能在 Android 的主執行緒，就是 UI 執行緒操作。所以會利用 Thread 或 AsyncTask 操作，類似這樣 :
``` java
// 新增一個執行緒來處理圖片 
new Thread() {
    @Override
    public void run() {
        super.run();
        // 遍步 folder 列表
        for (File folder : folders) {
            File[] files = folder.listFiles();

            // 遍步 file 列表
            for (File file : files) {

                //  篩選 png 檔案
                if (file.getName().endsWith(".png")) {
                    final Bitmap bitmap = getBitmapFromFile(file);

                    // 再回到 ui 執行緒添加圖片
                    getActivity().runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            imageCollectorView.addImage(bitmap);
                        }
                    });
                }
            }
        }
    }
}.start();
```

但是 RxJava 提供 Scheduler 這個控制器，能夠切換執行緒，這邊先了解 Scheduler 的各種方法 :

* Schedulers.immediate() : 直接在當下的執行緒運行，就等於不切換，是默認的方法
* Schedulers.newThread() : 像 new Thread() 一樣啟動一個新執行緒，並在裡面操作
* Schedulers.io(): 便於 I/O 操作的執行緒，與 `newThread()` 差不多，主要用於讀取文件、db等工作
* Schedulers.computation() : 計算時所使用的執行緒，要與 io 執行緒區別
* AndroidSchedulers.mainThread() : 指定回 Android UI 主執行緒運行

有了這些方法，就能夠在創建 Observable 時，使用 `subscribeOn()` 和 `observeOn()` 來進行切換控制。

* Observable.subscribeOn() : 指定 subscribe() 發生時所在的執行緒
* Observable.observeOn() : 指定 subscribe() 返回時 Observer 為了消費事件所在的執行緒

講了難懂，我們來用 RxJava 改造看看剛剛處理圖片的方法吧 :
``` java
// 從 folders 取出每個 folder 
// 再將每個 folder 中的 files 中的 file 各自取出
Observable.from(folders)
    .flatMap(new Func1<File, Observable<File>>() {
        @Override
        public Observable<File> call(File file) {
            return Observable.from(file.listFiles());
        }
    })

    // 過濾出檔名結尾為 .png 的檔案
    .filter(new Func1<File, Boolean>() {
        @Override
        public Boolean call(File file) {
            return file.getName().endsWith(".png");
        }
    })

    // 將讀取的每個 File 轉成 Bitmap 輸出
    .map(new Func1<File, Bitmap>() {
        @Override
        public Bitmap call(File file) {
            return getBitmapFromFile(file);
        }
    })

    // 規定以上這些 subscribe 中的工作，都要在 IO 執行緒 操作
    .subscribeOn(Schedulers.io())

    // 而我們希望最後返回 Bitmap 並顯示在 imageView 的工作，回到 mainThread 執行
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Action1<Bitmap>() {
        @Override
        public void call(Bitmap bitmap) {
            // 就是 call 裡面的工作在 mainThread 執行
            imageCollectorView.addImage(bitmap);
        }
    });
```
簡單的加入了 `subscribeOn()` 與 `observeOn()` 就讓執行緒能夠隨意切換，真棒 !

咦 ? 可是... 怎麼感覺程式碼變多了 ? 仔細看一遍，其實是邏輯變得簡單了呢 ! 程式碼中沒有任何迴圈什麼的，而且一條線的完成了整個工作，非常乾淨。重點是，若隔了好幾個月，突然要加入新的功能進來，用原本的方法寫的程式碼，又不愛加註解，可能要讀個幾分鐘才能搞懂吧 ! 而用 RxJava 的優勢就來了，一下就能搞懂自己當初在幹嘛，並且快速新增新功能進去 !


# 加入 Lambda Function
或許在 **「邏輯」** 與 **「程式碼」** 的簡潔，需要有所取捨。 Java 8 所新增的 Lambda 語法能夠讓 **程式碼** 變得更簡潔，但相對的會較看不出他的邏輯。 或許應該把 Lambda 的學習在寫一篇文章的 ... 不過先用看看，試著起到拋磚引玉的效果吧

## Setup
要使用 Java 8 的 Lambda 語法，需要在 app Gradle 中配置 (Android Studio 2.3)
``` gradle
android {
    defaultConfig {
    ...
        jackOptions {
            enabled true
        }
    }
    compileOptions {
        targetCompatibility 1.8
        sourceCompatibility 1.8
    }
}
```
其實自己也還沒有很認真學習過 Java 8 的各種寫法，所以這邊先點出一些常用的方法 :


## Lambda 語法
這是一般要設定 setOnClickListener 時的做法，可以看到 new View.OnClickListener() 之後的 onClick 方法，看起來太礙眼了，尤其是要定義一堆一樣的 Listener 時，版面之混亂 >_>

``` java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        Log.d(TAG, "Hello World!");
    }
});
```

所以 lambda 語法，提供了一種匿名(Anonymous) 表達式。因為要創建一個 OnClickListener 時，固定只使用到 onClick 方法，並且只要帶入參數 View ，所以就可以簡化成 ` (參數) -> { 工作方法(參數) } `
``` java
button.setOnClickListener(view -> {
   Log.d(TAG, "Hello World!");
});

// 還可以更簡化
button.setOnClickListener(view -> Log.d(TAG, "Hello World!"));

// 參數可以直接使用
button.setOnClickListener(view -> Log.d(TAG, view.getTag().toString()));
```

## lambda 方法引用 (method reference)
lambda 還能更簡化，這個方法就是方法引用 (method reference) ， 長這樣 `::` 。
用法就是，當你指定的 `工作方法` 中，那個參數跟你原本帶入的參數相同，就可以使用。
有點模糊，試試看 :
``` java
// 假如我已經創建好一個方法來執行偵錯，只要帶入 View 就抓出他的 Tag 值
private void show(View view) {
    Log.d(TAG, view.getTag().toString());
}

// lambda 就會知道這個方法可以直接使用
button.setOnClickListener(this::show);
```

## 實際在 RxJava 操作
學到這邊一定就知道， RxJava 有一堆 Action、Func 超煩的，來試試把他們簡化，一樣用剛剛處理圖片的流程 :

由於 map 與 subscribe 都已經有相對方法了，所以 Android Studio 還會問我們要不要改成 `方法引用` ，真貼心 !
``` java
Observable.from(folders)
                .flatMap(file -> Observable.from(file.listFiles()))
                .filter(file -> file.getName().endsWith(".png"))
                .map(this::getBitmapFromFile)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(this::showBitmap);
```

哇塞，剛剛有一定長度的程式碼，就被縮減為 7 行，而且還保留一定的邏輯在，好有成就感阿 !

---

# 結語
學會這篇文章的內容，差不多就算入門了 RxJava ，不過 RxJava 還有很多東西可以學習，像是 RxBinding 或是與 Retrofit 一起使用等。這些在邊實作邊學習吧 ! 現在重要的是多用 RxJava 開發，熟能生巧 !

# Reference

- [【Android】RxJava的使用（一）基本用法](http://www.jianshu.com/p/19cac3c5b106)
- [详细解析RxAndroid的使用方式](http://www.jianshu.com/p/6d1ef9f43cdc)
- [给 Android 开发者的 RxJava 详解 - 扔物线](http://gank.io/post/560e15be2dca930e00da1083#toc_15)
