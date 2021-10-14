---
toc: true
layout: post
title: 學習 Android Data Binding
description: 學習 Android Data Binding
categories: [Android, Architecture Pattern]
date: 2017-04-29 12:56:49
---

為了要在 Android 更方便的運作 NVVM 架構， Google 推出了 Data Binding 並且在 Android Studio 2.0.0 版本能夠正式的使用。

在還沒遇見 Data Binding 之前，例如要架構一個登入的 Activity ，必須要先去編寫 UI 的 XML ，並給每個需要與用戶互動的 View 一個 View ID 。建構好 UI 的部分後， 再去 java 編寫 findViewById() 、 setText() 、 setVisibility() 、 使用 Picasso 之類的第三方操作圖片⋯。

雖然我們可以利用 Butterknife 之類的 Library，降低大量重複煩躁的程式碼，但在更新時還是會同時干涉到 XML 與 java 。有了 Data Binding ，能夠進一步減輕開發的力氣，那就實際操作來學習看看吧。

# Setup

在 app 的 build.gradle 新增 dataBinding
``` xml
android {
    ....
    dataBinding {
        enabled = true
    }
}
```

# 最簡單的用法

## 定義 Model
``` java
public class User {
    public User(String name){
        this.name = name;
    }

    private String name;

    public String getName() {
        return name;
    }
}
```

## 定義 layout 檔
Data Binding 的 layout 檔與以前的寫法不同了。view 不在需要 ID ，不需要再回到 java 依序配對。這表示在未來新增新的 UI 時，會變得更方便。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<!-- 重點 1 -->
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- 重點 2 -->
    <data>  
        <variable
            name="user"
            type="com.sekaij.ddpractice.User"/>
    </data>

 	......
    <RelativeLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content">

 	<TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@{user.name}"     
                tools:text="username" />
                            <!-- 重點 3 -->
    </RelativeLayout>
</layout>
```

| 重點 |                                                                   |
| ---- | ----------------------------------------------------------------- |
| 1    | Data Binding 中的 layout 以 `<layout>` 作為最頂端的節點  |
| 2    | 在 `<data>` 內定義與這個 layout 有相關互動的物件            |
| 3    | 使用表達式 `@{}` 與物件產生連結。我們可以使用 tools 佈置介面 |

## 在 Activity 綁定數據
Data Binding 會依照 layout 檔自動產生 Binding 的 Class ，例如 `login_activity.xml` ，會產生一個名為 `(LoginActivity)Binding` 的 Class。

他代表了所有我們定義的綁定關係，所以必須在 inflate 創建他，後面在 recyclerView 也是如此。
``` java
public class MainActivity extends AppCompatActivity {

    private ActivityMainBinding binding; // 自動產生 binding 類別

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main);

        User user = new User("Jay");
        binding.setUser(user); // 自動根據 data 中的 variable 產生方法
    }
}
```

## 表達式
回到 `@{}` 表達式，除了可以利用如 `@{user.name}` 來綁定數據外，也可以加入一些簡單的邏輯運算。
``` xml
// 基本款
<TextView android:text="@{user.name}"/> 

// 可以加入 "三元運算子" 來判斷
<TextView android:text="@{user.age < 18 ? @string/redacted : user.name}"/>

// 也可以添加 View 來判斷 VISIBLE
<data> <import type="android.view.View"/> </data>
<TextView android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>

// 原本需要判斷是否為 null 的三元運算，也能夠使用合併運算符號 ?? 來判斷了
contact.lastName != null ? contact.lastName : contact.name

// 如果 lastName 為 null 就用 name ，不為空就使用自己
contact.lastName ?? contact.name
```

如果要引入資源時，則利用如 `@{@color/white}` 方式來引入。
``` xml
<... android:background="@{user.admin ? @color/colorAccent : @color/white}" />

<... android:padding="@{isBig ? @dimen/bigPadding : @dimen/smallPadding}" />
```

# 事件處理 (Event)

## 一般用法

首先先定義 click Event 的類別。
``` java
public class EventHandler {
    Context context;

    public EventHandler(Context context) {
        this.context = context;
    }

    // 因為綁定 view ， 所以需要設定參數 view
    public void onClicked(View view) {
        Toast.makeText(context, "Success ! ", Toast.LENGTH_SHORT).show();
    }
}
```
別忘了 在 binding 中設定 handler:
``` java
EventHandler handler = new EventHandler(this);
binding.setHandler(handler);
```

接著在 layout 檔定義 handler 的 variable ，並且代入 onclick
``` xml
<variable
    name="handler"
    type="com.sekaij.ddpractice.MainActivity.EventHandler"/>

// 記得使用 ::
// . 已經被棄用
<Button android:onClick="@{handler::onClicked}" />
```

也可以使用 java 8 提供的 lambda 表達式
``` xml
<Button android:onClick="@{(v) -> handler.onClicked(v)}" />

<!-- 可以不帶入 view -->
<!-- public void onClicked() {...} -->
<Button android:onClick="@{() -> handler.onClicked()}" />
```

## 帶入函數
``` java
User user = new User("Jay");
binding.setUser(user);

EventHandler handler = new EventHandler(this);
binding.setHandler(handler);

    ...

public void onButtonClicked(User user) {
    Toast.makeText(view.getContext(), "Success ! " + user.getName() + " 様", Toast.LENGTH_SHORT).show();
}
```

``` xml
<Button android:onClick="@{() -> handler.onButtonClicked(user)}" />

```

# 可觀測性 Observable
Data Binding 還能夠在數據更新時，通知 UI 變動， 只需將要更新變動的 object 繼承 BaseObservable 類別
``` java
public class User extends BaseObservable {
    private String name;

    public User(String name){
        this.name = name;
    }

    @Bindable
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
        notifyPropertyChanged(BR.name);
    }
}
```
BR 是編譯後產生的一個類別，會去尋找被標註 @Bindable 的 getter 方法，在 BR 中產生對應的值。
當數據更新後， `notifyPropertyChanged` 會通知系統 `BR.name` 已經變動，需要更新 UI 。


## ObservableFields
如果覺得要設定 getter 與 setter 很麻煩，可以設定 ObservableField 以及衍生的 ObservableBoolean 、 ObservableInt ⋯ 等。

``` java
public class ObservableUser {

    // 一定要加 public final
    public final ObservableField<String> name = new ObservableField<>();
    public final ObservableBoolean isAdmin = new ObservableBoolean();

    public ObservableUser(String name, boolean isAdmin) {
        this.name.set(name);
        this.isAdmin.set(isAdmin);
    }
}

// 定義
ObservableUser observableUser = new ObservableUser("Jay", true);

// 取得值
String name = observableUser.name.get();

// 設定值
observableUser.name.set("Wei");
```

## ObservableArrayMap
我們還可以使用 key-value 來存取數據，Data Binding 提供了 ObservableArrayMap 類別。
``` java
ObservableArrayMap<String, String> item = new ObservableArrayMap<>();

item.put("age", "20");
binding.setItem(item);
```

``` xml
<data>
<variable
    name="item"
    type="android.databinding.ObservableArrayMap&lt;String,String&gt;"/>
</data>
<!-- &lt; 與 &gt; 代替 < > -->
<!-- 雖然會顯示錯誤 但編譯正常 -->

<TextView android:text='@{item["age"]}' />
<!-- 使用 [] 讀取數據 -->
```


# BindingAdapter
除了基本的設定 setText 以外，當我們要設定圖片時怎麼辦，如果不是從 drawable 讀檔而是從 url ， 如果還想要使用 Picasso ， Glide 等第三方加載圖片時呢？ 這可以用 `BindingAdapter` 來解決！

``` java
public class CustomDataBindingAdapter {

    @BindingAdapter("picasso")
    public static void setImageUrl(ImageView view, String url) {
        Picasso.with(view.getContext()).load(url).into(view);
    }
}
```
``` xml
 <ImageView app:picasso="@{user.imgURL}" />
```

如此一來，我們就可以自己定義自己想要的 setter 了！

在 @BindingAdapter 註解的後面，也可以放入多個參數如 ：
``` java
@BindingAdapter({"imageUrl", "error"})
public static void loadImage(ImageView view, String url, Drawable error) {
    Glide.with(view.getContext()).load(url).error(error).into(view);
}
```
``` xml
<ImageView 
    app:imageUrl="@{url}"
    app:error="@{@drawable/ic_launcher}"/>
```


# RecyclerView 運用

Data Binding 節省了 ViewHolder 的程式碼，不需要再定義 findViewById ，不需要 holder.location = ... ，只需要使用 binding 類別給予的 inflate 進行綁定。

## 定義 Observable 物件與 ClickHandler
```java
public class TimeZone {
    public final ObservableField<String> location = new ObservableField<>();
    public final ObservableField<String> time = new ObservableField<>();

    public TimeZone(String loc, String time) {
        this.location.set(loc);
        this.time.set(time);
    }
}

public interface TimeZoneClickHandler {
    void onTimeZoneClick();
}
```

## 定義 item layout
``` xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="timeZone"
            type="com.sekaij.ddpractice.TimeZone" />
        <variable
            name="handler"
            type="com.sekaij.ddpractice.TimeZoneClickHandler"/>
    </data>

    <LinearLayout
        android:layout_margin="16dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="@{() -> handler.onTimeZoneClick()}">

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:textColor="@android:color/darker_gray"
            android:text="@{timeZone.location}"
            tools:text="location" />


        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="@{timeZone.time}"
            tools:text="time" />

    </LinearLayout>
</layout>
```

## 定義 ViewHolder
``` java
public class TimeZoneViewHolder extends RecyclerView.ViewHolder {

        private RecyclerViewItemBinding binding;

        public TimeZoneViewHolder(RecyclerViewItemBinding binding) {
            super(binding.getRoot());
            this.binding = binding;
        }

        public void bindTo(TimeZone timeZone) {
        	binding.setTimeZone(timeZone);

            // binding.setVariable(BR.timeZone, timeZone); 
            // 如果在 xml 定義的 variable 是與他人共用相同的 data ，需要使用這個方法

            binding.executePendingBindings();
        }

        public void setHandler(final Context context, final TimeZone timeZone) {
            binding.setHandler(new TimeZoneClickHandler() {
                @Override
                public void onTimeZoneClick() {
                    String location = timeZone.location.get();
                    String time = timeZone.time.get();
                    Toast.makeText(context, location + " : " + time, Toast.LENGTH_SHORT).show();
                }
            });
        }
    }
```

這邊需要注意的是 `executePendingBindings` 這個方法，當我們設定數據時，數據綁定要等到下一個動畫幀才會觸發，所以我們才需要這個方法，強制執行更新 UI。

## 定義 Adapter
``` java
public class TimeZoneRecyclerAdapter extends RecyclerView.Adapter<TimeZoneRecyclerAdapter.TimeZoneViewHolder> {

    private ArrayList<TimeZone> timeZoneList = new ArrayList<>();
    private Context context;

    public TimeZoneRecyclerAdapter(Context context, ArrayList<TimeZone> timeZoneList) {
        this.context = context;
        this.timeZoneList = timeZoneList;
    }

    @Override
    public TimeZoneViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        LayoutInflater layoutInflater = LayoutInflater.from(parent.getContext());
        RecyclerViewItemBinding binding = RecyclerViewItemBinding.inflate(layoutInflater, parent, false);
        return new TimeZoneViewHolder(binding);
    }

    @Override
    public void onBindViewHolder(TimeZoneViewHolder holder, int position) {
        holder.bindTo(timeZoneList.get(position));
        holder.setHandler(context, timeZoneList.get(position));
    }

    @Override
    public int getItemCount() {
        return timeZoneList.size();
    }
```


# 結語

Data Binding 讓我們能夠在 xml 定義一些表達式，讓 UI 跟 Activity 降低更多耦合，但也不代表我們能把全部的邏輯都放到 xml ， 然後不用再寫 java 。

例如我們不應該在 xml 使用一個發送網路訊息的功能。
``` xml
<ImageView android:click="@{webservice.sendMoneyAsync}"/>
```

我們應該只處理一些關於 UI 介面的事情。
``` xml
<ImageView android:click="@{presenter.onSendClick}"/>
```

到此，還有很多 Data Binding 與 DI 的運用，以及將 Data Binding 帶進更棒的 NVVM 架構！

# Reference

- [棉花糖給Android 帶來的Data Bindings](https://news.realm.io/cn/news/data-binding-android-boyar-mount/)
- [Android Data Binding 系列(一) -- 详细介绍与使用](https://goo.gl/iJhLb1)

第一個網站為 Android Data Binding 發表的詳細講解，有詳細的講述 Data Binding 背後的運行，所以在這篇文章就不詳細探討，而是實作 Data Binding 的技術。













