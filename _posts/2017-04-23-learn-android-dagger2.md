---
toc: true
layout: post
title: 學習 Android 依賴注入框架 Dagger2
description: 學習 Android 依賴注入框架 Dagger2
categories: [Android, Dependency Injection]
date: 2017-04-23 14:06:51
---

從上一篇 [學習 Android 依賴注入 Dependency Injection (DI)]({% post_url 2017-04-22-learn-android-dependency-injection %}) 我們知道，單純的手動使用 DI 進行編寫，會發生不斷依賴反而讓程式碼變得更複雜的問題。所以這次要來學習前人的智慧 DI 框架的使用，我選擇使用 Dagger2 作為學習的框架。不僅僅因為 Dagger2 的名聲響亮， Dagger2 除了被推崇為最好解決 Android DI 的框架，也適合進行單元測試，更棒的是網路上已經有大量的學習文章了！

## 使用 Dagger2 流程

<img src='{{site.baseurl}}/images/markdown/learn-android-dagger2/flow.jpg' width=666>

## @Inject
回到上次慘不忍睹的程式碼，不斷的創建造成開發效率變得很低。

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
Dagger2 使用註解 (Annotations) 來標註 Client 所依賴的 dependency ，這個註解的名字稱作 @Inject 。

```java
public class ModuleA {

    private ModuleB moduleB;

    @Inject
    public ModuleA(ModuleB moduleB) {
        this.moduleB = moduleB;
    }
}
```
同樣的，也用註解 @Inject 來標註 dependency 的 構造函數 (Constructor)
讓 Client 的 dependency 與 dependency 的 Constructor 產生無形的連結。

```java
public class ModuleB {

    private ModuleC moduleC;
    private ModuleD moduleD;

    @Inject
    public ModuleB(ModuleC moduleC, ModuleD moduleD) {
        this.moduleC = moduleC;
        this.moduleD = moduleD;
    }
}
```



## @Module
另一種讓 dependency 之間產生無形連結的方法，就是透過 Module 。
Module 就像一個工廠一樣，我們統一在 Module 裡生產相關的 dependency。

```java
@Module
public class ApplicationModule {

    public ModuleA provideModuleA(ModuleB moduleB) {
        return new ModuleA(moduleB);
    }

    public ModuleB provideModuleB(ModuleC moduleC, ModuleD moduleD) {
        return new ModuleB(moduleC, moduleD);
    }

    public ModuleC provideModuleC(ModuleE moduleE) {
        return new ModuleC(moduleE);
    }

    public ModuleD provideModuleD() {
        return new ModuleD();
    }

    public ModuleE provideModuleE() {
        return new ModuleE();
    }
}
```
Module 就是一個 Class ，不過這個 Class 需要用 @Module 來標註，讓 Dagger2 能夠判斷。
創建好 Module 後，就可以在 Module 中生產每個 dependency 了。

### @Provides
既然 Module 是一個 Class ，那在 Module 中就可以有負責生產 dependency 的 Method ，也可以有負責做其他事情的 Method 。

這種專門為了生產 dependency 的 Method 稱作 Provider ，要利用 @Provides 標註。

```java
@Module
public class ApplicationModule {

    @Provides
    public ModuleA provideModuleA(ModuleB moduleB) {
        return new ModuleA(moduleB);
    }

    @Provides
    public ModuleB provideModuleB(ModuleC moduleC, ModuleD moduleD) {
        return new ModuleB(moduleC, moduleD);
    }

    @Provides
    public ModuleC provideModuleC(ModuleE moduleE) {
        return new ModuleC(moduleE);
    }

    @Provides
    public ModuleD provideModuleD() {
        return new ModuleD();
    }

    @Provides
    public ModuleE provideModuleE() {
        return new ModuleE();
    }
}
```
@Provides 有一個好處，今天要使用 ModuleA 的時候，系統看到 ModuleA 需要 ModuleB ，就會去找其他 Provider ，看有沒有生產 ModuleB 的 Provider ，然後創建一個 ModuleB 給 ModuleA 使用。
如果 ModuleB 還需要其他 dependency ，系統就會繼續找下去，直到所有 dependency 都被滿足。

### Context ?
那假如今天有一個 Provider 需要 Context 怎麼辦呢，例如 SharedPreferences 。

```java
@Module
public class ApplicationModule {

    private final Context context;

    public ApplicationModule(Context context) {
        this.context = context;
    }

    @Provides
    public Context provideContext() {
        return context;
    }

    @Provides
    public ModuleD provideModuleD(Context context) {
        return new ModuleD(context);
    }
}
```
之前說過 Module 也是 Class 的一種，那我在產生 Module 時，就可以利用 Module 的 Constructor 引入 Context 囉！


## @Component
知道了 @Module 和 @Inject 的用法，那要怎麼讓那些被 @Inject 與 @Module 的 dependency 產生直接的連結，然後在 Activity 上被更簡單的創建使用？

這就是 Component 大展身手的時候了， Component 扮演了 Injector 的角色，就像注入者一般把那些已經產生無形連結的 dependency 注入 Client 中。


Component 跟 Module 不同了， 不是 Class 而是 Interface 。當然，也要用 @Component 標註一下。

```java
@Component
public interface ApplicationComponent {
}
```

### Component 與 Inject
今天如果 ModuleA、B、C ... 都已經做好 Constructor 的 Inject 時，那要怎麼在 MainActivity 實現 ModuleA 呢。

```java
@Component
public interface ApplicationComponent {
    void inject(MainActivity mainActivity);
}
```
首先要在 Component 裡定義一個 inject 的方法，意思就是告訴 Component 到 MainActivity 中，去找所有被 @Inject 的物件。

```java
public class MainActivity extends AppCompatActivity {

    @Inject
    ModuleA moduleA; // Inject 的物件不能設為 private 不然 Component 會找不到

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ApplicationComponent component = DaggerApplicationComponent.builder().build();
        component.inject(this);
    }
}
```
dagger2 會對所有經過 @Component 標註過的 interface 進行處理，自動產生一個實現了這個 interface 的 Class ， Class 的名字就是 Component 的名字前面加上“Dagger”，例如 ApplicationComponent 就會變成 DaggerApplicationComponent 。

找到被 @Inject 註解的 ModuleA 之後，Component 繼續去找 ModuleA 的 dependency 被標註的 Constructor ，對應就是被 @Inject 標註的 ModuleB Constructor ，並創建 ModuleB 返回給 ModuleA 。一樣，如果 ModuleB 也有相對應的 dependency ， Component 也會不斷的找下去。


### Component 與 Module
實際操作時，可能會創建多個 Module 負責不同的 dependency ，也有可能有多個 Component 。
所以要怎麼讓 Component 知道去哪裡找 dependency 呢？
Dagger2 規定，我們必須要指定 Component 負責的 Module 是誰，就在 @Component 註解的後面定義。

```java
@Component(modules = {ApplicationModule.class})
public interface ApplicationComponent {
    ModuleA moduleA();
}
```
如果 MainActivity 需要 ModuleA ，那我們就在 Component 裡定義一個返回 ModuleA 的方法。
這樣一來， Component 在產生 ModuleA 的時候，就會到 Module 裡面去尋找 ModuleA 相關的 dependency ，一樣創建返回給 ModuleA。

```java
public class MainActivity extends AppCompatActivity {

    private ModuleA moduleA;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ApplicationComponent component = DaggerApplicationComponent.builder().build();
        moduleA = component.moduleA();
    }
}
```
呼叫 Module 中的 Provider 除了用這種方式，還可以用原本的 Inject 的方式操作。

```java
@Component
public interface ApplicationComponent {
    void inject(MainActivity mainActivity);
}
```
Component 會到 MainActivity 找所有被 @Inject 的物件，然後回到 Module 中使用對應的 Provider ，在返回給物件。

# 結語
到這邊都只是用了簡單的 Inject、Module、Component 來完成 DI ，還有好多 Dagger2 的功能，像是 Singleton、Scope ，怎麼樣分配 Component 的工作，還沒有提到。希望在大量的練習之後，能夠更深入去分析 Dagger2。

# Reference

- [小創作- Android單元測試（六）：使用dagger2來做依賴注入，以及在單元測試中的應用](http://chriszou.com/2016/05/10/android-unit-testing-di-dagger.html)
- [Android：dagger2讓你愛不釋手-基礎依賴注入框架篇](http://www.jianshu.com/p/cd2c1c9f68d4)