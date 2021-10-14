---
toc: true
layout: post
title: 學習 Android 依賴注入 Dependency Injection (DI)
description: 學習 Android 依賴注入 Dependency Injection (DI)
categories: [Android, Dependency Injection]
date: 2017-04-22 16:11:19
---

之前學到的幾種 Android 設計架構，都是為了要讓程式碼簡化，使得程式可讀性變高，以及更順利、簡單的進行單元測試。而在 [Android單元測試（五）：依賴注入，將mock方便的用起來](http://chriszou.com/2016/05/06/android-unit-testing-di.html) 這篇文章學習單元測試時，發現依賴注入 (Dependency Injection，DI)能夠讓單元測試變得更容易，甚至可以讓 App 的架構變得更乾淨。究竟 DI 是何方神聖，身為一個入門開發者一定要來學一下。

# 什麼是依賴注入

根據 [聊聊 Android 中的依賴注入](http://android.jobbole.com/82386/) 這篇文章敘述，原來 DI 只是一種 設計模式 (design pattern) ，而且還有許多的框架可以使用。 DI 的目的就是讓開發者能夠寫出”低耦合“的程式碼，不但能更輕鬆的進行單元測試，也幫助維持整個 App 的架構。畢竟如果一款要生存長久的 App ，開發的時間就會越長，往往測試的效率就會慢慢地降低。所以善用 DI 來改善設計，增加可維護性、可拓展性、降低耦合，試著讓程式簡潔優雅一些吧。

# 依賴注入的概念

## 什麼是 Dependency ?
在 DI 的觀念裡有兩個角色 : Client 、 Dependency ，如果在 App 裡面，有 A Class 用到了 B Class ，那麼 A 就是 Client ，而 B 則是 Dependency 。

```java
public class ModuleA {
   private ModuleB moduleB;

   public ModuleA() {
      moduleB = new ModuleB();  // A 產生了對 B 的 dependency
   }
}
```

## 什麼是 Injection ?
上面的程式碼中， B 在外部已經創建好， 接著就像被注入 (Inject) 一般 set 到 Client 的 A裡面，這就算是 DI 的一種方式。但是在上面的例子中 A 和 B 之間還是存在高度耦合，不算是一個很好的 Injection ，一般常用的方法，是透過 client 的 Constructor 將 dependency 傳入。

```java
public class ModuleA {
   private ModuleB moduleB;

   public ModuleA(ModuleB moduleB) {
      this.moduleB = moduleB;  // 將 B 作為 A 的 Constructor 參數傳入
   }
}
```

現在，A Class 不需要知道如何去實現 B ，只要任何繼承了 B 的 Class 都可以傳入，降低兩者間的耦合。
到這邊就是 DI 的概念，簡單。


# 依賴注入的問題
習得了 DI 技能馬上就去打 Boss ，發現了一個天大的問題。如果你手動一個一個一個的新增 dependency ，你會發現、你會訝異，所有的 dependency 從最頂端的 Client 一直延伸開來。

```java

// 如果原本的 ModuleB 也需要兩個 dependency C & D
public class ModuleB {
   private ModuleC moduleC;
   private ModuleD moduleD;

   public ModuleB(ModuleC moduleC, ModuleD moduleD) {
      this.moduleC = moduleC;
      this.moduleD = moduleD;
   }
}
```

而 ModuleC 還需要 Module E !

```java
public class ModuleC {
   private ModuleE moduleE;

   public ModuleC(ModuleE moduleE) {
      this.moduleE = moduleE;
   }
}
```

那可憐 Activity 要呼叫 ModuleA 時就會變成這樣 。･ﾟ･(つд`ﾟ)･ﾟ･

```java
public class MainActivity extends AppCompatActivity {
	private ModuleA moduleA;

	@Override
    protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ModuleD moduleD = new ModuleD();
        ModuleE moduleE = new ModuleE();

        ModuleC moduleC = new ModuleC(moduleE);
        ModuleB moduleB = new ModuleB(moduleC, moduleD);

        moduleA = new ModuleA(moduleB);
	}
}
```

有夠亂啊 !!! 
還好，前人們在很早以前使用 DI 時早就出現了這樣的問題，所以早就有許多解決使用 DI 問題的框架。


# 依賴注入的框架

現在我們的問題是，如果有越來越多的 dependency 被我製造時，這些 dependency 可能會在不同的 client class 被 new 出來， 不但變得更複雜了，而且還重複了一堆程式碼。

而在 Java 的領域中，已經有很多框架幫助我們解決問題，例如最近流行的一種框架叫作 Dagger2 。

這些框架幫助我們建立一個類似 dependency 的工廠，所有的 dependency ，還是 dependency 的 dependency ，都要統一在這個工廠裡生產。所有要用到這些 dependency 的 client 就去這個工廠取得，而且 client 只需要知道他要用的 dependency 是誰，不需要知道他要的 dependency 又用了哪些 dependency ， 框架系統會自動幫我們判斷。

所以接著來學學怎麼使用 Dagger2 吧！看下一篇: [學習 Android 依賴注入框架 Dagger2]({% post_url 2017-04-23-learn-android-dagger2 %})

# Reference

- [Android單元測試（五）：依賴注入，將mock方便的用起來](http://chriszou.com/2016/05/06/android-unit-testing-di.html)
- [聊聊 Android 中的依賴注入](http://android.jobbole.com/82386/)