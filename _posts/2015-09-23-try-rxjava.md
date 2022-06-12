
---
title: 小试RxJava
date: 2015-09-23T13:55:08+08:00
categories: [撸代码, Android]
tags: [笔记]
---

RxJava是NetFlix推出的[Reactive Extensions][0](简写为Rx)框架的Java语言实现。[Reactive Extensions][0]实际是以现在非常火的响应式编程范式的为基础的一种变体，基于响应式编程加入了一些函数式编程的元素在里面。

Rx系列发展到现在，基本覆盖了所有主流编程语言，包括C#、Java和JavaScript等等。在Android平台上，也有很多出名的公司基于RxJava开发了许多衍生库，并在其App中大量使用，据我了解有：Square(Jake Wharton大神出版的[RxBinding][1])、Trello(推出了[RxLifecycle][2])和国内的大众点评。

在这篇文档，主要是写下RxJava的用法。

## 基本概念

官方对Rx的表述是：

> Rx是一个函数库，让开发者可以利用可观察序列和LINQ风格查询操作符来编写异步和基于事件的程序，使用Rx，开发者可以用Observables表示异步数据流，用LINQ操作符查询异步数据流， 用Schedulers参数化异步数据流的并发处理，Rx可以这样定义：Rx = Observables + LINQ + Schedulers。

而Rx的响应式编程概念以我的理解，我觉得可以简单的概括为“观察者模式+管道操作符”，而实际上，RxJava的操作过程就是： "Observable -> Operators -> Subscriber"。显而易见的，`Observable`为模型中的目标对象，`Subscriber`为观察者，而`Operators`负责目标对象发出的信息给观察者们的分发，也就是管道。其中`Operators`是Rx最关键的一部分，`Operators`不仅可以分发信息，而且可以对信息数据进行各种处理，甚至可以把信息回馈到另外的目标对象中。这样做，可以完成`Subscriber`和`Observable`的解耦。

## 使用方法

根据上面的表述，RxJava的使用方法非常简单，也就是我们定义好工作目标Observable和观察者Subscriber，然后把他们组合在一起就可以了，如果有必要对中间的流程进行处理，则在中间加入相应的Operators。

不过，首先我们要在项目中引入RxJava库。

### 在Android工程中引入RxJava

随着ADT停止更新，现在绝大部分Android开发都是使用Android Studio或者Intellij Idea了，这两者都是基于Gradle来构建Android工程的，那么我们可以直接引入RxJava Maven Repo即可在项目中使用RxJava了：

```groovy
dependencies {
    compile 'io.reactivex:rxjava:1.0.14'
}
```

另外，虽然RxJava最低支持到Java 6，但是总所周知，Java是个纯面向对象语言，它并没有支持函数式编程，也只有在Java 8的时候，Java 8引入几个函数式编程元素(Lambda表达式、默认方法和函数式引用)。而Rx是个函数式编程框架库，在Java 7之前使用的话，代码看起来会非常别扭，所以最好是在Java 8的环境中使用。不过Android现在官方支持只到Java 7，没支持Java 8，要想在Android中使用Java 8，则必须加入第三方插件。

### 最简单的模式

> Observable -> Subscriber

是最为简单模型，这里我们假设从接收到的网络请求中获取到一串用户列表，我们要把用户列表显示到页面上，用RxJava可以这样写：

```java
List<User> users; /* 获取到的用户列表 */
UserAdapter adapter; /* 页面Adapter */

/* 中间环节忽略 */

/* 创建observable */
Observable<List<User>> observable = Observable.create(new Observable.OnSubscribe<List<User>>() {
    @Override
    public void call(Subscriber<List<User>> subscriber) {
        subscriber.onNext(users);
        subscriber.onCompleted();
    }
});

/* 创建subscriber */
Subscriber<List<User>> subscriber = new Subscriber<List<User>>() {
    @Override
    public void onCompleted() {
	Log.d("test", "completed!");
    }

    @Override
    public void onError(Throwable e) {
	Log.d("test", "error!");
    }
    @Override
    public void onNext(List<User> args) {
        adapter.setDataSet(args);
    }
};

// 订阅
observable.subscribe(subscriber);
```

值得注意的一点就是如果Observable没有被Subscriber订阅的话，Observable的内部代码是不会被执行的，这有个好处就是提供了一个控制手段给我们，来控制何时执行这个事件。

实际上，这样一个简单的流程，写了这么多行代码，所以RxJava提供了比较简便的方法：

比如，Observable可以这样写：

```java
// 创建observable
Observable<List<User>> observable = Observable.just(user);
```

`Observable.just()`是RxJava提供的一个语法糖，主要是用简单的传递数据。在RxJava中，共有这几种创建Observable的方法：

* `just( )`， 将一个或多个对象转换成发射这个或这些对象的一个Observable
* `from( )`，将一个Iterable, 一个Future, 或者一个数组转换成一个Observable
* `repeat( )`， 创建一个重复发射指定数据或数据序列的Observable
* `repeatWhen( )`，创建一个重复发射指定数据或数据序列的Observable，它依赖于另一个Observable发射的数据
* `create( )`，使用一个函数从头创建一个Observable
* `defer( )`，只有当订阅者订阅才创建Observable；为每个订阅创建一个新的Observable
* `range( )`，创建一个发射指定范围的整数序列的Observable
* `interval( )`，创建一个按照给定的时间间隔发射整数序列的Observable
* `timer( )`，创建一个在给定的延时之后发射单个数据的Observable
* `empty( )`，创建一个什么都不做直接通知完成的Observable
* `error( )`，创建一个什么都不做直接通知错误的Observable
* `never( )`，创建一个不发射任何数据的Observable

方法虽然很多，但是在以页面展示的功能需求中，我们还是只用了这几个方法：`create`、`repeat`、`repeatWhen`和`defer`，所以只要掌握这几种用法就可以了，其余的需要的时候才查文档。关于创建Observable的文档在[这][3]。

同样，如果我们不关心subscriber是否结束（onComplete())或者发生错误(onError()),subscriber的代码可以简化为：

```java
// 创建subscriber
Action1<List<User>> subscriber = new Action1<List<User>>() {
    @Override
    public void call(List<User> args) {
        adapter.setDataSet(args);
    }
};
```

我们直接把创建和订阅连接起来，完整的代码如下：

```java
Observable.just(users).subscribe(new Action1<List<User>>() {
    @Override
    public void call(List<User> args) {
        adapter.setDataSet(args);
    }
});
```

这样看起来已经更简洁了，但是如果使用Java 8的话，代码会更加好看：

```java
Observable.just(users).subscribe(args -> adapter.setDataSet(args));
```

### 加入Operators

很多时候，我们需要针对处理过的事件做出响应，而不仅仅是Observable产生的原始事件。最明显的例子就是网络请求回来的数据往往是JSON数据，我们还要对此进行序列化成相应的业务操作才可以传递给前台，这里就需要引入operator来处理JSON数据。

所以之前一节的代码实际上缺少解析JSON的部分，完整的步骤应该先获取JSON，再序列化然后传给页面，所以Observable传递的应该为JSON数据，而subscriber接收的却是业务对象，中间必须加入Operator处理序列化，代码如下：

```java
Observable.just(json).map(new Func1<JSONObject, List<User>>() {
    @Override
    public String call(JSONObject json) {
        return User.constructionFrom(json);
    }
}).subscribe(args -> adapter.setDataSet(args));
```

这里使用了名为`map`operator，它的作用很简单，就是接收一个事件，并返回处理后的事件。Func1的第一个泛型参数表示输入类型，第二个泛型参数表示返回类型。

同样，使用Java 8的语法会更加简洁： 

```java
Observable.just(json)
        .map(json -> User.constructionFrom(json))
        .subscribe(args -> adapter.setDataSet(args));
```

RxJava中的Operator一共有二十几个，而且自己也可以自定义Operators，这正式RxJava比一般的异步框架优秀的地方，由于自带的Operators数量过多，这里就不一一讲述了，需要的可以查下[文档][4]。

### 任务订阅

前面缺少了RxJava的一个细节，就是实际上执行Observable.subscribe()时，它会返回一个Subscrition,它代表了Observable和Subscriber之间的关系。我们可以通过Subscrition解除Observable和Subscriber之间的订阅关系，并立即停止执行整个订阅链，比如要取消上述显示用户列表操作，我们可以这样做：

```java
Subscription subscription = Observable.just(json)
        .map(json -> User.constructionFrom(json))
        .subscribe(args -> adapter.setDataSet(args));
subscription.unsubscribe();
Log.d("test", "isSubscribed = " + subscription.isUnsubscribed());
```

### 多线程操作

Android的多线程机制是UI渲染跑在Main Thread上的，也就是说主线程不能阻塞，不然App就会出现卡顿，所以我们一般都把业务逻辑处理放在后台线程进行。在RxJava中，你可以通过subscribeOn()来指定Observer的运行线程，通过observeOn()指定Subscriber的运行线程。这两个方法都是operator，因此它们可以像所有operator那样作用于任何的Observable。比如上面的例子，我们可以把JSON解析放在后台线程，然后在Main Thread中更新用户信息：

```java
Observable.just(json)
        .map(json -> User.constructionFrom(json))
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())     /* 这里用了RxAndroid */
        .subscribe(args -> adapter.setDataSet(args));
```

值得注意的是RxJava自带的`Schedulers.io`和`Schedulers.compute`两种异步多线程Scheduler开启的线程的级别都是与Main Thread一样的，这样后台线程占用的资源大的话，主线程还是会卡的，所以最好还是自己自定义一个Scheduler。

### MVP

RxJava中的Observable、Operators和Subcribers是各自独立的，所以我们可以很容易的针对不同场景复用不同部分的代码。

下面就是自己写的一个Demo中显示系统所有APP INFO的代码(Kotlin)的简单示范：

Model层，负责获取系统内安装的所有App数据:

```kotlin
interface AvailableAppModel {
    fun data(): Observable<List<AppInfo>>
}

class AvailableAppModelImpl(val ctx: Context) : AvailableAppModel {

    override fun data(): Observable<List<AppInfo>> {
        return Observable.create { subscriber ->
            val packs = ctx.packageManager.getInstalledPackages(PackageManager.GET_ACTIVITIES)
            val apps = ArrayList<AppInfo>()
            for (pack in packs) {
                if (pack.applicationInfo.flags and ApplicationInfo.FLAG_SYSTEM == 0) {
                    apps.add(genAppInfo(ctx, pack))
                }
            }
            subscriber.onNext(apps)
            subscriber.onCompleted()
        }
    }
}
```

Presenter层，负责传递数据到View层，而传递用的就是RxJava，其中Model层的数据请求放在后台线程执行:

```kotlin
interface AvailableAppPresenter {
    fun bindView(view: AvailableAppView)
    fun bindModel(model: AvailableAppModel)
    fun init(args: Bundle?)
}

class AvailableAppPresenterImpl(val ctx: Context) : AvailableAppPresenter {
    private var appView: AvailableAppView? = null
    private var appModel: AvailableAppModel? = null

    override fun bindView(view: AvailableAppView) {
        appView = view
    }

    override fun bindModel(model: AvailableAppModel) {
        appModel = model
    }

    override fun init(args: Bundle?) {
        appModel!!.data()
                .subscribeOn(AndroidRxPlugin.workerThread)
                .observeOn(AndroidRxPlugin.mainThread)
                .subscribe(appView!!.renderData())    /* 只有订阅了Subscriber才执行获取数据请求 */
    }
}
```

View层:

```kotlin
public class MainActivity : AppCompatActivity(), AvailableAppView {

    val rvApps: RecyclerView by bindViewById(R.id.rv_apps)
    var adapter: AppAdapter? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        rvApps.layoutManager = LinearLayoutManager(this)
        adapter = AppAdapter(this)
        rvApps.adapter = adapter

        val presenter = AvailableAppPresenterImpl(this)
        val model = AvailableAppModelImpl(this)
        presenter.bindView(this)
        presenter.bindModel(model)
        presenter.init(savedInstanceState)
    }

    override fun renderData(): Subscriber<List<AppInfo>> = Subscribers.create { l -> adapter!!.applist = l }
}
```

总体来说，RxJava是非常不错的，响应式编程的概念的引入起码解决了Android回调不美观的问题。不过响应式编程和函数式编程的概念可能一时比较难接受，还需要段时间来适应。

[0]: http://reactivex.io
[1]: https://github.com/JakeWharton/RxBinding
[2]: https://github.com/trello/RxLifecycle
[3]: http://reactivex.io/documentation/operators.html#creating
[4]: http://reactivex.io/documentation/operators.html
