---
title: Android下的任务与Activity栈
date: 2013-11-04T20:28:15+08:00
categories: [撸代码, Android]
tags: [笔记]
---

我们都知道，通过Intent启动Activity有两种形式:

1. 显式的指向某个Activity  

```java
  Intent intent = new Intent(this, Activity1.class);
  startActivity(intent); 
```

2. 隐式的通过设置Intent的动作启动Activity

```java
  Intent intent = new Intent(Intent.ACTION_VIEW);
  intent.setData(Uri.parse("http://www.google.com"));
  startActivity(intent);
```

Activty是Android四大组件中负责与用户交互的部件，Activity承担了大量的显示和交互工作，从某种角度上将，我们看见的应用程序就是许多个Activity的组合。

## Task Stack

为了让这许多Activity协同工作而不至于产生混乱，Android平台设计了一种堆栈机制用于管理Activity，其遵循先进后出的原则，系统总是显示位于栈顶的Activity，从逻辑上将，位于栈顶的Activity也就是最后打开的Activity，这也是符合逻辑的。这个栈也可以叫做Task Stack，因为一个Task Stack里的Activity是可以属于不同的Application的。例如：你想在发送短信时，拍一张照并作为彩信发出去，这时你首先停留在短信应用程序的的Acitivity上，然后跳转到Camera应用程序的Activity上，当完成拍照功能后，再返回到短信应用程序的Activity。这实际上是两个Android Application协同合作后完成的工作，但为了更好的用户体验，Android平台加入了Task这么一种机制，让用户没有感觉到应用的中断，让用户感觉在一“应用程序”里就完成了想完成的工作。

通过下图可以更清晰的理解Application、task、Activity三者之间的关系：

![Application-task-Activity](http://kescoode.qiniudn.com/android-activity-task-0.png)

`值得注意点的时候，一个系统里可以有很多的Task Stack，当后台的Task过多的时候，系统可能会去除栈底的Activity，释放多余的内存。`

## 保存Activity状态

当Task Stack中的Activity返回前台的时候，可能已经是被系统释放点重新创建的，为了用户的操纵信息得到保留，我们最好重写[onSaveInstanceState()][0]方法。

## Managing Tasks

我们在平常写程序的时候，Activity的启动方式保持默认就已经够用了。但是遇到特定场合，我们可以通过对启动方式的更改来修改Activity在Task中的运行状态。

Activity的启动状态我们可以在AndroidManifest.XML中定义，抑或直接通过Intent传相应的数值对。

- 操作manifest文件，Activity状态设置是其节点launchMode属性定义，共有四种形式：

	1. "standard"  
	默认方式，Activity可以创建多次，分布在不同的Task Stack中，有相应不同的实体。
  
	2. "singleTop"  
	通过此方法启动的Activity如果在栈顶的话，当再次收到启动的Intent，将不会再创建新实例，而是直接执行[onNewIntent()][1]方法。Activity可以有多个实例，在每个Task Stack中除非不在栈顶，在接受到启动的Intent的时候都不会创建新实例。  
	比如一个Task Stack:A->B->C->D，如果是Standard模式，当收到D的启动Intent时，Stack:A->B->C->D->D，如果是SingleTop模式,Stack:A->B->C->D。而当收到的是B的启动Itent时，Stack:A->B->C->D->B,因为B不在栈顶。
	
	3. "singleTask"  
	这种模式下，Activity只能存在一个实例，当再次收到启动的Intent时，直接执行[onNewIntent()][1]方法。
	
	4. "singleInstance"
	类似于singleTask，不同的地方是，Activity只能有一个实例，而且该实例的Task Stack只能有这个Activity，也就是说通过该模式启动的Activity在Task Stack中即是栈顶也是栈底。

- Intent通过setFlags或者addFlags方法添加数值对，默认方式是不用做任何操作的，除此之外还有三种形式：

	1. FLAG_ACTIVITY_NEW_TASK
	启动一个新的Task Stack，当包括Activity的Task已经存在的时候，这个Task将会直接在前台显示，类似与singleTask。
	
	2. FLAG_ACTIVITY_SINGLE_TOP
	当这个Intent启动的前台的Activity，将不会创建新实例，而是直接执行[onNewIntent()][1]，类似于singleTop。
	
	3. FLAG_ACTIVITY_CLEAR_TOP
	如果启动的Activity在Task Stack中，其余的Activity都会被弹出释放，不会创建新实例，而是执行[onNewIntent()][1]。


[0]:http://developer.android.com/reference/android/app/Activity.html#onSaveInstanceState(android.os.Bundle)
[1]:http://developer.android.com/reference/android/app/Activity.html#onNewIntent(android.content.Intent)
