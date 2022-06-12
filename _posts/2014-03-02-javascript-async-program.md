---
title: JQuery异步编程
date: 2014-03-02T20:48:00+08:00
categories: [撸代码, JavaScript]
tags: [笔记]
---

JavaScript异步编程通常的方法是指定一个回调函数，当操作结束之后，自动执行回调函数。但是这样有个不好的一点，也就是容易造成回调函数嵌套，比如下面一个搞笑的情况：

```
              ......
			   ......
                ......
                           }
                         }
                       }
                     }
                   }
                 }
               }
......
......
```

为了避免这种情况的出现，业界现在有几种编程范式。如CommonJS提出的Promise模式，就是个不错的解决方案。jQuery的deferred对象就是jQuery的Promise范式回调函数解决方案。

## deferred对象与传统回调函数的对比

这里用AJAX请求做示列，传统写法：

```javascript
$.ajax({
　　　　url: "test.html",
　　　　success: function(){
　　　　　　alert("哈哈，成功了！");
　　　　},
　　　　error:function(){
　　　　　　alert("出错啦！");
　　　　}
　　});
```

deferred对象写法：

```javascript
$.ajax("test.html")
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
```

这里，实际上`.done`方法相当与`success`方法，`.fail`方法相当于`error`方法。从这里可以看出，deferred对象对于整个代码结构来说会更清晰明了，同时更可以采用链式写法，`$.ajax('test.html').done(...).fail(...).done(...)`，操作起来是非常方便的。

## 同时为多个异步操作指定回调函数

jQuery提供`$.when`方法来为多个事件指定一个回调函数，写法如下：

```javascript
$.when($.ajax("test1.html"), $.ajax("test2.html"))
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
```

当两个请求都成功后会执行`.done`方法，失败则会执行`.fail`方法，实际上，`$.when`返回的就是一个deferred对象。

## Deferred对象方法

初步了解deferred对象的操作后，我们可以看下deferred包含的一些方法：

1. `$.Deferred`生成一个deferred对象。
2. `deferred.done`指定操作成功时的回调函数
3. `deferred.fail`指定操作失败时的回调函数
4. `deferred.promise`没有参数时，返回一个新的deferred对象，该对象的运行状态无法被改变；接受参数时，作用为在参数对象上部署deferred接口。
5. `deferred.resolve`手动改变deferred对象的运行状态为"已完成"，从而立即触发`.done`方法。
6. `deferred.reject`这个方法与`deferred.resolve`正好相反，调用后将deferred对象的运行状态变为"已失败"，从而立即触发`.fail`方法。
7. `$.when`为多个操作指定回调函数。
8. `deferred.then`有时为了省事，可以把`.done`和`.fail`合在一起写，这就是`.then`方法。
9. `deferred.always`这个方法也是用来指定回调函数的，它的作用是，不管调用的是`deferred.resolve`还是`deferred.reject`，最后总是执行。

## 自定义的回调函数

有了上面的deferred方法后，我们可以这样定义自己的异步操作：

```javascript
var wait = function(dtd){
　　　　var dtd = $.Deferred(); //在函数内部，新建一个Deferred对象
　　　　var tasks = function(){
　　　　　　alert("执行完毕！");
　　　　　　dtd.resolve(); // 改变Deferred对象的执行状态
　　　　};

　　　　setTimeout(tasks,5000);
　　　　return dtd.promise(); // 返回promise对象
　　};
　　$.when(wait())
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
```
