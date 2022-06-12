---
title: 用Travis CI给Android项目部署Github Release
date: 2015-10-15T14:29:53+08:00
categories: [撸代码, Android]
tags: [笔记]
---

时至今日，互联网软件的开发和发布，已经形成了一套标准流程，其中一个重要部分就是持续集成(CI)。  

平时我工作中CI用得比较多的地方是按照git tag做应用渠道分发打包。最近写了一个拖拽手势Xposed插件[SlideBack-Xposed][0]，除了[Xposed Module Repo][1]外，我需要一个提供APK对外下载的地方。由于Google Play审核机制收紧了，单纯Xposed插件收录比较麻烦，而我又不想放到国内渠道上，于是我把注意力放到Github Release上。

Github Release是Github推出的一个帮助开发者发布Repo最终软件包给用户的功能。Github Release是根据其Repo的Git标签来排序的，而且目前Github还没有限制Github Release的存储大小，不需担心空间不够用，所以一般情况下，已经可以了。同时，我们利用Travis CI来自动根据我们打的Git Tag来构建APK并Push到自己的Github Release上。

## 启用Travis CI

Travis CI是比较流行的开源持续构建平台，与Github结合比较紧密，对Github上的开源Repo是免费的，私有Repo收费（Circle CI也不错，而且对Github私有Repo也是免费的）。我们首先在Travis CI上打开对自己项目的Hook开关，那么当你每次Commit Push的时候，CI就会自动运行了。

![travis-enable-project](http://7mnom1.com1.z0.glb.clouddn.com/travis-enable-project.png)

## 基础构建脚本

Travis CI使用YAML文件作为构建脚本，以最简单Android项目构建APK为例，只需在项目根目录创建`.travis.yml`文件：

```yaml
language: android
android:
  components:
    - build-tools-22.0.1
    - android-23
    - extra-android-m2repository
    - extra-android-support
script:
  - ./gradlew assembleRelease
```

然后`git add . && git commit && git push`即可。这里解释下，`.travis.yml`几个关键的Tag：

1. 因为Repo是Android项目，所以构建语言`language`选择`android`。
2. 选择了Android项目后，就在`android`中的`components`的tag中设置Android项目需要的依赖。
3. Travis CI编译Android实际上也是调用项目中的构建脚本的。现在大部分Android项目都是用Gradle构建的，如果是要打Release版本的APK包，在`script`加入`./gradlew assembleRelease`就行了。

不过，实际上，单纯在CI上面打包，其实是没什么用处的，因为Travis CI每次任务完成之后，就会把所有生成的文件清掉，所以我们要把生成的APK传到Github Release上。

## 发布Github Release

Travis CI默认支持发布到Github Release上，不需要配置别的脚本，相应的`YAML`配置Tag是这样的：

```yaml
deploy:
  provider: releases
  api_key: "GITHUB OAUTH TOKEN"
  file: "FILE TO UPLOAD"
  skip_cleanup: true
  on:
    tags: true
```

值得注意的一点就是`skip_cleanup`这个Tag要设置为`True`，不这样做的话，Travis CI在部署之前就会清空生成的APK文件，那样你就什么都得不到了。`api_key`是部署到Github Release的凭证，需要开发者自己生成。如果不想那么麻烦的话，可以使用Travis CI提供的`travis`命令行工具。

`travis`是用Ruby写的，安装需要系统配有Ruby 1.9以上版本：

```shell
gem install travis -v 1.8.0 --no-rdoc --no-ri
```

安装完成后，在根目录执行：

```shell
travis setup releases
```

按照相应的提示，就可以生成相应的部署配置了。完整的`.travis.yml`文件如下：

```yaml
language: android
android:
  components:
    - build-tools-22.0.1
    - android-23
    - extra-android-m2repository
    - extra-android-support
script:
  - ./gradlew assembleRelease
deploy:
  provider: releases
  api_key:
    secure: [生成的token]
  file: app/build/outputs/apk/app-release.apk
  skip_cleanup: true
  on:
    tags: true
```

这样，以后在你Repo打git tag并把它Push到Github上后，Travis CI就会自动帮你构建一个Github Release了。

![slideback-github-release](http://7mnom1.com1.z0.glb.clouddn.com/slideback-github-release.png)

等等，难道这样就结束了吗？当然不了，这只是单纯把APK发送到Github Release上去。但是构建APK过程中还有很多问题，比如你在CI构建Release版本的APK的时候，要用到自己签名的密钥，对吧。我们当然不能把密钥的信息公开，那么该怎么做呢？

## 隐藏签名密钥信息

Travis CI是通过Android项目中的build.gradle的信息来构建apk的，自然我们通过配置build.gradle脚本来签名apk也是很正常的事。但是在公开项目中暴露签名密钥信息是不安全的，有个简单的方法是，让gradle在CI构建的时候从CI的环境变量中读取密钥信息，这样可以降低点风险。

首先，我们先在`build.gradle`配置签名选项文件：

```groovy
android {
    signingConfigs {
        release {
            storeFile file(".kesco.keystore")
            storePassword System.getenv("KEYSTORE_PASS")
            keyAlias System.getenv("ALIAS_NAME")
            keyPassword System.getenv("ALIAS_PASS")
        }
    }
}
```

像这样的话，gradle在签名的时候会自动从系统读取`KEYSTORE_PASS`、`ALIAS_NAME`和`ALIAS_PASS`三个环境变量。

然后我们在Travis CI上设置`KEYSTORE_PASS`、`ALIAS_NAME`和`ALIAS_PASS`这三个环境变量：

![](http://7mnom1.com1.z0.glb.clouddn.com/travis-env.png)

另外，Travis CI在构建Android项目时，默认使用Oracle JDK，但是当我设置CI的环境变量后，构建会报错，所以就把JDK换为OpenJDK了：

```yaml
jdk: openjdk7
```

如果还想更保密点的话，可以将签名密钥keystore文件加密一下，等CI启动的时候再解密，这样会安全些。

以上就是我折腾[SlideBack-Xposed][0]项目时总结出来的经验，下面是我项目中完整的`.travis.yml`文件：

```yaml
language: android
jdk: openjdk7
android:
  components:
    - build-tools-22.0.1
    - android-23
    - extra-android-m2repository
    - extra-android-support
git:
  submodules: false
before_install:
  - sed -i 's/git@github.com:/https:\/\/github.com\//' .gitmodules
  - git submodule init && git submodule update
script:
  - ./gradlew assembleRelease
before_deploy:
  - mv app/build/outputs/apk/app-release.apk app/build/outputs/apk/slideback.apk
deploy:
  provider: releases
  api_key:
    secure: [Github Token]
  file: app/build/outputs/apk/slideback.apk
  skip_cleanup: true
  on:
    tags: true
```

[0]: https://github.com/kesco/SlideBack-Xposed
[1]: http://repo.xposed.info/module/com.kesco.xposed.slideback
