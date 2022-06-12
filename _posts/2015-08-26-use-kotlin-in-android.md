---
title: 用Kotlin编写Android应用
date: 2015-08-26T17:16:13+08:00
categories: [撸代码, Android]
tags: [笔记]
---

最近，Kotlin在外面比较火，外面的呼声也很大。我也是今年4月份看到Jake Wharton的这篇[文章][0]后才开始关注的。后来，因为工作项目中，引用了ikew0ng巨巨的[SwipBackLayout][1]拖拽关闭页面库，而作者本人又不继续维护了，所以我试着用Kotlin参考[SwipBackLayout][1]，自己撸了一个轮子[SlideBack][2]，发现用Kotlin来开发Android还是比较舒服的。

最近，开发团队内技术分享，我做了个简单的[PPT][4]，就是简单介绍下Kotlin的。

## 语法

Kotlin的语法非常简洁，熟悉Java的开发者可以快速上手。而且JetBrain系的IDE上还可以一键把已有的Java代码转换成Kotlin代码(不得不说一键傻瓜式操作系列真的非常人性化=_=)，有点像Groovy，开发者可以一开始以Java的风格写Kotlin代码，然后慢慢转成Kotlin自己的风格。详细的语法可参考[Kotlin官方教程][3]，这里做个简短介绍。

### 声明对象

Kotlin的一切变量都是对象，所以没有Java那样的基本类型。Kotlin声明对象有两种类型：

1. 可变对象

```kotlin
var x = 5                       // 类型推断为`Int`型
var b: String = "Hello"         // String型
```

2. 不可变对象，Kotlin没有final关键字，而且不存在静态变量

```kotlin
val x = 5                       // 不可变`Int`型变量
x += 1                          // 编译器会报错
```

从代码片段可以看出，Kotlin不需要在每行结束加分号。

### 区分非空类型

在Kotlin中，nullable对象和nullable对象是严格区分的，甚至在编译期解决了不少潜在的空指针问题。声明变量时，对变量的类型都是默认非空的，如果要允许变量为空，必须在定义类型后面加个`?`：

```kotlin
var a: String = "hello"         // a字符串不可为空
var b: String? = "hello"        // b字符串可以为空
a = null                        // 编译器会报错
b = null                        // OK!!!
```

而且，在对有可能为空的对象进行操作时，编译器会提示Warning。同时，Kotlin提供类似Ruby和CoffeeScript的语法糖：

```kotlin
var b: String? = "hellp"
b?.length()                     // 如果b不为空对象，则取b的长度
```

### 智能类型转换

在Kotlin中，进行强制类型转换可以使用`as`关键字，但有可能会抛出异常：

```kotlin
if (c is String) {              // Kotlin使用`is`关键字判断对象类型
    c.length()
}
```

在上面的例子中，如果`c`是一个String对象，则在if块中，可以直接使用String的方法，编译器会智能的帮你识别出c在if-blcok里面是一个String对象。

Kotlin也提供一个"安全"的类型转换方式：
```kotlin
val d: String? = c as String?
/* 或者 */
val d: String? = c as? String
```

### 流程控制

#### `if`
Kotlin的`if`表达式与Java的一样，只是Kotlin中没有三目表达式，所以`if`和`else`可以这样写：

```kotlin
val a = 1
val b = 2
val max = if (a > b) a else b   // 类似Java的: int max = a > b?a:b;
```

#### `when`
Kotlin用`When`表达式来替代Java的`Switch`：

```kotlin
when (x) {
  1 -> print("x == 1")
  2 -> print("x == 2")
  else -> { // Note the block
    print("x is neither 1 nor 2")
  }
}
```

#### `for`

Kotlin的`for`表达式和Java的`foreach`一致。

```kotlin
for (i in array.indices)
  print(array[i])
```

#### `while`

`while`表达式和Java的一样。

```kotlin
while (x > 0) {
  x--
}

do {
  val y = retrieveData()
} while (y != null)     // 与Java不同
```

### 函数

与Java不同的是，Kotlin的函数是一等成员，不需要在类内定义，是可以脱离类存在的，而且Kotlin是不支持类静态方法的。

```kotlin
fun hello():Unit {      
    print("hello")
}

fun add(a: Int, b: Int):Int{
    return a + b
}
```

Kotlin没有void关键字，函数都是要返回对象的，所以如果没东西返回的时候，函数后要声明Unit类型(M10版本之后就默认不需要了)。

如果函数比较简单可以放在一行的话，甚至可以这样：

```kotlin
fun add(a: Int, b: Int) = a + b
```

在这种情况下，函数默认返回最后计算的结果。

#### 扩展类的函数

通常开发中，我们往往要对提供的API类进行扩展，增加一些方法，如果是Java的话，要想这样做，则声明一个继承该API的子类。Kotlin采取了C#的办法，可以直接扩展类的方法：

```kotlin
fun Fragment.findViewById(id: Int) = this.getView.findViewById(id)
```

从而不需要衍生出一堆子类或者辅助工具类。

那么问题来了，如果扩展的类里面本来就有这个同名方法，但类对象调用这个同名方法的时会出现什么情况呢？答案是： 如果类里面就有这个方法，Kotlin就会调用原来的方法，而不调用扩展方法。

利用这个特性，Kotlin的扩展函数可以提供旧版本API兼容。比如自Android API 16之后，View提供了`setBackground`方法，原来的`setBackgroundDrawable`则被标记为过时的了，如果要在旧Android手机上使用该API，我们可以这样写：

```kotlin
fun View.setBackground(background: Drawable) = this.setBackgroundDrawable(background:)
```

这样，在旧的手机上，APP就可以用自定义的`setBackground`的Wrapper，而在高版本的手机上APP会调用原生的`setBackground`方法。

#### Lambda表达式

Kotlin引入了Lambda表达式，而且Kotlin的Lambda表达式支持Android平台：

```kotlin
view.setOnClickListener({ toast("Hello world!") })
```

这样，我们就可以不用写那么多监听器对象了。
]

### 类

Kotlin的类是这样声明的：

```kotlin
class User(val id: Int,val name: String) { // 只有一个构造方法是，可以这样声明
    init {
        print("Constructor $id : $name")
    }
    // Nothing   
}

val user = User(1, "Kesco")
```

从上面代码片段可以看出，Kotlin在调用构建方法时，会先调用`init`代码块内的代码，而且构建类对象的时候，是不需要`new`关键字的。

那么，如果有多个构造方法怎么办呢？在Kotlin的类内，构造方法名都是规定为`constructor`的：

```kotlin
class User { // 只有一个构造方法是，可以这样声明
    var _id: Int = 0
    var _name: String = ""

    constructor(id: Int, name: String) {
        _id = id
        _name = name
    }

    constructor(name: String): this(0, name) {
    }

    init {
        print("Constructor $id : $name")
    }
    // Nothing   
}

val user = User(1, "Kesco")
```

Kotlin的类默认是final的，也就是不可继承，如果让类可继承，则要带有`open`关键字声明：

```kotlin
open class User(val id: Int, val name: String) {
    // Nothing
}
```

虚类Kotlin与Java一样，都是用`abstract`关键字声明，当有`abstract`关键字的时候，就不需要带有`open`了：

```kotlin
abstract class User() {
    // Nothing
}
```

#### Getter和Setter

Kotlin的Setter和Getter编码风格与C#类似：

```kotlin
class User {
    private var _id: Int
    var id: Int
        get() = _id
        set(value) {
            _id = value
        }
}
```

#### Data Class

Kotlin的类可以申明`data`关键字，相当与专用与存储数据的Pojo类：

```kotlin
data class User(val id: Int = 0, val name: String = "")
```

而且Data Class可以进行这样的操作：

```kotlin
val jane = User(1, "Jane")
val (id, name) = jane
println("$name id is $id")
```

#### 内部类

Kotlin同样支持内部类，但是Kotlin的内部类是默认不带有外部类的引用的，也就是说默认的Kotlin内部类都是静态的。要想内部类带有外部类的引用，要在内部类声明上加入`inner`关键字：

```kotlin
class User(val id:Int, val name: String) {
    inner class School(val name: String) {
        // Nothing
    }
}
```

### 接口

Kotlin的接口和Java的类似，而且还支持Java 8的默认方法：

```kotlin
interface MyInterface {
    fun bar()
    fun foo() {
        print("foo")
    }
}
```

## Android工程中配置Kotlin

在Android项目中使用Kotlin非常简单，而且Kotlin可以和Java混编，所以完全部分功能用Kotlin开发，部分功能用Java开发。

首先，确保Android Studio或者Intellij Idea安装了Kotlin插件。

然后，在项目Module的`build.gradle`上声明：

```groovy
apply plugin: 'kotlin-android'
```

接着，添加Kotlin依赖：

```groovy
dependencies {
    compile 'org.jetbrains.kotlin:kotlin-stdlib:0.1-SNAPSHOT'
}
```

最后，添加Kotlin源码文件夹即可：

```groovy
android {

    ...

    sourceSets {
        main.java.srcDirs += 'src/main/kotlin'
    }
}
```


[0]: https://docs.google.com/document/d/1ReS3ep-hjxWA8kZi0YqDbEhCqTt29hG8P44aA9W0DM8/edit?hl=zh-CN&forcehl=1
[1]: https://github.com/ikew0ng/SwipeBackLayout
[2]: https://github.com/kesco/SlideBack
[3]: http://kotlinlang.org/docs/reference/
[4]: http://kesco.github.io/kotlin_intrudction
