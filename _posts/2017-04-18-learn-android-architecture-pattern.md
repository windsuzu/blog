---
toc: true
layout: post
title: 練習在 Android 設計上的 MVC, MVP, MVVM 架構
description: 練習在 Android 設計上的 MVC, MVP, MVVM 架構
categories: [Android, Architecture Pattern]
date: 2017-04-18 22:39:15
---

# 前言

一個人單打獨鬥設計 App ，可能不需要太講究架構，而是注重 App 的完整性與功能性，但是太凌亂的設計開發，反而會降低開發的效率，在未來的維護也會造成很大的麻煩。

層次分明、架構清楚漂亮的代碼，能夠實現低耦合的模組化，不但看起來賞心悅目，也讓自己在開發測試上變得更容易。在未來，如果有緣與他人合作，熟悉各種架構寫法的人，肯定也是吃香的。

為此，這次要來學習怎麼在 Android 上建立幾種軟體架構。當然，我們不能為了設計而設計，一個簡單的 App ，卻為了設計而花費更多成本，反而本末倒置。


<img src='{{ site.baseurl }}/images/markdown/learn-android-architecture-pattern/ui.jpg' style="float: right; margin: 15px" width=180>

# 先認識 M - V - X 

Model View Controller (MVC) 已經是一種很廣泛流行的架構模式，近幾年也被運用到組織 Android App 上。 隨後衍生的 Model View Presenter (MVP) & Model View ViewModel (MVVM) ，兩種不同的架構也在各種開發者的推崇下，分為好幾派。

不過每種架構在開發上都有好有壞，要如何在適當的時機運用適當的架構，讓開發變得更得心應手，才我們真正要學習的，所以這次藉由這篇 [Read MVC vs. MVP vs. MVVM on Android](https://realm.io/news/eric-maxwell-mvc-mvp-and-mvvm-on-android/) ， 試著利用不同的架構，寫出一個 " 圈叉遊戲 " ！

# Android 中的 MVC
在 Model View Controller 的設計理念中， Model 處理數據及邏輯， View 顯示邏輯結果， Controller 則負責起到兩者的橋樑，藉此來達到分離 Model 與 View ，做到高聚合與低耦合的模組化。

## Model
Model 在 "圈叉遊戲" 中，要為我們處理 **資料 + 狀態 + 邏輯運算** ，簡單來說就是這個 App 的大腦。 Model 不被綁定在 View 或 Controller ，也因此他還能具備 reuseable 的特質。


## View
View 在此則是擔任將 Model 視覺化的要角。 View 需要去執行 UI 相關的工作，或當 user 與 app 互動時與 Controller 溝通。 他不會參與到太多有關邏輯運算的工作，也不需知道 user 與 app 互動時的狀態為何。


## Controller
Controller 就像膠水一樣把整個 App 聯繫在一起。當 View 告知 Controller 說 User 按下按鈕時， Controller 就要決定怎麼與 Model 互動，產生新的邏輯結果，並且反映更新 View 的介面。 通常在 Android 裡， Activity 與 Fragment 就是擔任 Controller 的角色。

<img src='{{ site.baseurl }}/images/markdown/learn-android-architecture-pattern/mvc.png' style="float: right;margin: 15px" height="250" width="250">

## 實作

<p style="color:red"/>Model 負責遊戲中三個重要元素
- Board : 運作遊戲的邏輯 (重新開始、換誰下、下在哪裡⋯⋯等)
- Cell : 定義遊戲內每一個 cell 的值
- Player : 定義玩家 O , X

<p style="color:blue"/>View 負責顯示 OX 遊戲的介面，與 menu 中的 reset 按鈕
<p style="color:green"/>Controller 則擔任與 View 、 Model 互動的角色

```java
public class TicTacToeActivity extends AppCompatActivity {
    private Board Model;

    /* 被引用的 View 物件 */
    private ViewGroup buttonGrid;
    private View winnerPlayerViewGroup;
    private TextView winnerPlayerLabel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.tictactoe);

        winnerPlayerLabel = (TextView) findViewById(R.id.winnerPlayerLabel);
        winnerPlayerViewGroup = findViewById(R.id.winnerPlayerViewGroup);
        buttonGrid = (ViewGroup) findViewById(R.id.buttonGrid);

        Model = new Board();
    }

    /* 綁定 reset() 到 menu reset 按鈕 */
    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.action_reset:
                reset();
                return true;
            default:
                return super.onOptionsItemSelected(item);
        }
    }

    // 當 cell 被 user 點擊時觸發， 我們更新 Model 並等待回傳值 Player
    // 如果這一步其中一個人贏，則顯示"贏家提示"的 View 
    // 否則就將這一步的Player mark 到 cell 裡 
    public void onCellClicked(View v) {

        Button button = (Button) v;

        String tag = button.getTag().toString();
        int row = Integer.valueOf(tag.substring(0, 1));
        int col = Integer.valueOf(tag.substring(1, 2));

        Player playerThatMoved = Model.mark(row, col);

        if (playerThatMoved != null) {
            button.setText(playerThatMoved.toString());
            if (Model.getWinner() != null) {
                winnerPlayerLabel.setText(playerThatMoved.toString());
                winnerPlayerViewGroup.setVisibility(View.VISIBLE);
            }
        }

    }

    /* 把"贏家提示"的 View 隱藏及清空 Board ，並告訴 Model 要 restart game 了 */
    private void reset() {
        winnerPlayerViewGroup.setVisibility(View.GONE);
        winnerPlayerLabel.setText("");

        Model.restart();

        for (int i = 0; i < buttonGrid.getChildCount(); i++) {
            ((Button) buttonGrid.getChildAt(i)).setText("");
        }
    }

}
```

## 結論
MVC 有效的分離了 View 與 Model ，我們因此可以更簡單的測試 Model ， 也不需要針對 View 進行測試。但在 Controller 的部分還有一些問題:
- 靈活度低 : Controller 跟 View 緊密耦合，一但更新了 View 也必須回到 Controller 修改。
- 可測試性 : Controller 綁定了 Android APIs ，很難進行單元測試。
- 維護性低 : 隨著不斷的開發，更多的代碼會被放在 Controller 當中，造成臃腫的現象。


# Android 中的 MVP
MVP 將 Controller 換成了 Presenter ， 意味著要降低 Controller 與 View 的緊密耦合， 讓 Activity 不再臃腫。


## Model
與 MVC 一樣，繼續處理 App 的 **資料 + 狀態 + 邏輯運算** 。


## View
既然 View 與 Activity 的關係密不可分， MVP 就將 Activity 與 Fragment 也視為了 View 的一部分，並且實作了 View Interface 讓 Presnter 能夠透過 Interface 與 View 產生互動，進而能夠實現單元測試。


## Presenter
Presenter 的工作基本上與 Controller 大同小異，只差在 Presenter 不再綁定 View ，解決了在 MVC 中所遇到的測試性與靈活度問題。

<img src='{{ site.baseurl }}/images/markdown/learn-android-architecture-pattern/mvp.png' style="float: right;margin: 15px" height="250" width="250">

## 實作
<p style="color:red"/>Model 與 MVC一樣負責 App 的資料邏輯
<p style="color:blue"/>View 將 Activity 帶進來，並新增了 View Interface
<br/>*View Interface 幫助 Presenter 與 View(Activity) 進行互動，也就可以模擬 Activity 的行為對 Presenter 進行單元測試*
<p style="color:green"/>Presenter 比起 Controller 的寫法更簡單了，不需要一邊處理UI的事情，一邊忙著執行各種動作。

```java
public interface TicTacToeView {
    void showWinner(String winningPlayerDisplayLabel);
    void clearWinnerDisplay();
    void clearButtons();
    void setButtonText(int row, int col, String text);
}
```

```java
public class TicTacToePresenter implements Presenter {

    private TicTacToeView view;
    private Board model;

    public TicTacToePresenter(TicTacToeView view) {
        this.view = view;
        this.model = new Board();
    }

    // 這裡把 Activity 的 Lifecycle 引進， 是從我們 implement Presenter Interface 而來
    public void onCreate() { model = new Board(); }
    public void onPause() { }
    public void onResume() { }
    public void onDestroy() { }

    // 當用戶選擇 cell 時， Presenter 只收到 (row,col)的資訊 ，不需要再在意 View
    public void onButtonSelected(int row, int col) {
        Player playerThatMoved = model.mark(row, col);

        if(playerThatMoved != null) {
            view.setButtonText(row, col, playerThatMoved.toString());

            if (model.getWinner() != null) {
                view.showWinner(playerThatMoved.toString());
            }
        }
    }

    // 需要 reset 時， Presenter 只需下達命令
    public void onResetSelected() {
        view.clearWinnerDisplay();
        view.clearButtons();
        model.restart();
    }
}
```

## 結論
MVC → MVP 之後， 原本 Activity 的邏輯處理跳到了 Presenter ， Activity 變為了 View 的角色，建立 UI 與 Presenter 的連結，最主要的是，View 與 Model 變得不直接互動了，而我們新增的 View Interface ，也更有利於單元測試運作。
現在我們可以單獨測試 Presenter 的邏輯，因為他不屬於任何 View ，而且也讓我們能夠跟不同的 View 進行協同只要 View 有 implement TicTacToeView 。
但在 Presenter 的部分還有一些問題:
- 維護性低 : Presenter 與 Controller 一樣，是負責一些附加的邏輯運算。一旦 App 需要不斷演進，開發者就會發現 Presenter 越來越臃腫，代碼量越來越大。


# Android 中的 MVVM
MVVM 中的 VM 是 ViewModel 的縮寫，透過了 [Data Binding on Android](https://developer.android.com/topic/libraries/data-binding/index.html) 的技術，實現更方便的測試跟模組化，也大量減少為了連結 View 與 Model 的代碼。


## Model
一樣。

## View
View 被綁定為 Observable variables ，與 ViewModel 可以雙向的互動。


## ViewModel
Data Binding 減輕原本 MVP 中 Presenter 要與 Model 和 View 互動的職責， ViewModel 的工作只需接管 Presenter 剩下的工作，包裝 Model 與 Observable Data 給 View 。


<img src='{{ site.baseurl }}/images/markdown/learn-android-architecture-pattern/mvvm.png' style="float: right;margin: 15px" height="250" width="250">

## 實作

<p style="color:red"/>Model 還是負責 App 的資料邏輯
<p style="color:blue"/>View 利用 Data Binding 的方式與 ViewModel 進行溝通
> 在此必須要去學習一下 Android 最新的 Data Binding
 可以查看 [學習Android Data Binding](https://windsuzu.github.io/2017/04/29/learn-android-databinding/)

<p style="color:green"/>ViewModel 負責一些與介面邏輯運算，完成 View 與 Model 的交互

```java
public class TicTacToeViewModel implements ViewModel {

    private Board model;

    // 這些 observable variables 與 view 已經綁定在一起
    // 能夠在 ViewModel 更新的同時迅速的更新，並且快速反映在 view 上
    public final ObservableArrayMap<String, String> cells = new ObservableArrayMap<>();
    public final ObservableField<String> winner = new ObservableField<>();

    public TicTacToeViewModel() {
        model = new Board();
    }

    // 現在這些動作將會被 view 直接呼叫
    public void onClickedCellAt(int row, int col) {
        Player playerThatMoved = model.mark(row, col);
        cells.put("" + row + col, playerThatMoved == null ? 
                                                     null : playerThatMoved.toString());
        winner.set(model.getWinner() == null ? null : model.getWinner().toString());
    }

    public void onResetSelected() {
        model.restart();
        winner.set(null);
        cells.clear();
    }
}
```

再來看看 view 的 xml 如何使用 View Binding:

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <import type="android.view.View" />
        <variable name="viewModel" type="com.acme.tictactoe.viewmodel.TicTacToeViewModel" />
    </data>
    <LinearLayout...>
        <GridLayout...>
            <Button
                style="@style/tictactoebutton"
                android:onClick="@{() -> viewModel.onClickedCellAt(0,0)}"
                android:text='@{viewModel.cells["00"]}' />
            ...
            <Button
                style="@style/tictactoebutton"
                android:onClick="@{() -> viewModel.onClickedCellAt(2,2)}"
                android:text='@{viewModel.cells["22"]}' />
        </GridLayout>

        <LinearLayout...
            android:visibility="@{viewModel.winner != null ? View.VISIBLE : View.GONE}"
            tools:visibility="visible">

            <TextView
                ...
                android:text="@{viewModel.winner}"
                tools:text="X" />
            ...
        </LinearLayout>
    </LinearLayout>
</layout>
```

# 結論

MVVM 在測試上變得更加的方便了，因為已經沒有任何綁定在 view元件 身上。在測試時，只需要驗證 observable variables 的設定是否正確，並且在更新時有適當的反映在 view 上。
但 MVVM 並不是萬能，將 data 綁定在 XML 中會有一些問題存在， XML 無法執行單元測試，所以有可能在運行時才發現錯誤，所以要多使用 android 提供的 tools 提高開發的效率。

# Reference

- [MVC vs. MVP vs. MVVM on Android](https://realm.io/news/eric-maxwell-mvc-mvp-and-mvvm-on-android/)
- [Android App的设计架构：MVC,MVP,MVVM与架构经验谈](https://www.tianmaying.com/tutorial/AndroidMVC)
- [框架模式 MVC 在Android中的使用](http://blog.csdn.net/feiduclear_up/article/details/46363207)
- [Android中的MVP \| Rocko's blog](https://rocko.xyz/2015/02/06/Android%E4%B8%AD%E7%9A%84MVP/)