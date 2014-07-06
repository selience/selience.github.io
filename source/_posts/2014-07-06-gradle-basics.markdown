---
layout: post
title: "Gradle Basics"
date: 2014-07-06 11:10:35 +0800
comments: true
categories: 
---
自Android Studio发布以来，我逐步的将工作空间从Eclipse迁移过来，对于老伙伴Eclipse也越来越嫌弃。也难怪，Eclipse运行效率和体验一直为广大码农所诟病，连基本的列表滚动都不够平滑，而Android Studio在各方面都达到了优秀，极具科技感的ui也牢牢地抓住了广大Android码农的心。

![](/media/2014-07-06-gradle-basics/android-studio.png)

<!--more-->
重点在于`Android Studio`默认采用`Gradle`编译项目，而不是我们所熟知的`Ant`，在[Gradle User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide)中能够看到Google给出的理由:
>Gradle is an advanced build system as well as an advanced build toolkit allowing to create custom build logic through plugins.
>
Here are some of its features that made us choose Gradle:
>
* Domain Specific Language (DSL) to describe and manipulate the build logic.
* Build files are Groovy based and allow mixing of declarative elements through the DSL and using code to manipulate the DSL elements to provide custom logic.
* Built-in dependency management through Maven and/or Ivy.
* Very flexible. Allows using best practices but doesn’t force its own way of doing things.
* Plugins can expose their own DSL and their own API for build files to use.
* Good Tooling API allowing IDE integration

`Gradle`是基于`Groovy`语言的，`Groovy`是一种运行于`JVM`的动态语言，语法与`Java`类似，并且可以在`Groovy`项目中引入`Java`代码块。这使得我大`Java`程序员可以低成本的定制编译逻辑。

---
Gradle文件结构
===
``` groovy 
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:0.4'
    }
}
apply plugin: 'android-library'

dependencies {
    compile files('../libs/android-support-v4.jar')
}

tasks.withType(Compile) {
    options.encoding = "UTF-8"
}

android {
    compileSdkVersion 17
    buildToolsVersion "17"

    defaultConfig {
        minSdkVersion 14
        targetSdkVersion 14
    }
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

        instrumentTest.setRoot('tests')
    }
}
```

以上是一个基本的`build.gradle`文件，整体结构分为三个主要的部分：
1. buildscript { ... } 这一块指定了编译环境，比如所依赖的gradle版本
2. apply plugin: 'android-library' 指定使用的插件，此文件指定的插件为'android-library'表明该项目是一个Lib项目，普通项目需指定为apply plugin: 'android'
3. android { ... } 项目参数，包含编译的SDK版本号，源码位置等。

---
依赖
===
常见的依赖有以下三种形式：
``` groovy
compile project(':ListViewAnimationLib') //对其他Lib项目的依赖
compile files('../libs/umeng_sdk.jar')  //jar包依赖
compile 'com.jakewharton.timber:timber:1.1.+' //从服务器抓去依赖包
```

---
多项目
===
`Gradle`对于`Android Lib Project`也提供了良好的支持，假设有一个项目其结构如下:

```
MyProject/
 + app/
 + libraries/
    + lib1/
    + lib2/
```
我们可以指定三个项目，`Gradle`将以如下方式命名这几个项目:
```
:app
:libraries:lib1
:libraries:lib2
```
每个项目会包含一个`build.gradle`文件声明该项目的编译过程，而在根目录需要额外添加一个`settings.gradle`文件，用于声明项目。
```
MyProject/
 | settings.gradle
 + app/
    | build.gradle
 + libraries/
    + lib1/
       | build.gradle
    + lib2/
       | build.gradle

```

`settings.gradle`的内容非常简单：
```
include ':app', ':libraries:lib1', ':libraries:lib2'
```

随后我们需要在`:app`这个项目中指名所依赖的项目:
```
dependencies {
    compile project(':libraries:lib1')
    compile project(':libraries:lib2')
}
```
---
签名&混淆
===
直接看代码吧
``` groovy
 signingConfigs {
        fuubo {
            storeFile file("../RefacTech.keystore")
            storePassword "pwd"
            keyAlias "key"
            keyPassword "pwd"
        }
    }

    buildTypes {
        release {
            runProguard true
            proguardFile 'proguard-project.txt'
            signingConfig signingConfigs.fuubo
        }
    }
```

---
ProductFlavors
===
这个功能可以用来打渠道包，以下配置可以在一次编译过程中生成beta 和 offical两种包，使用不同的manifest编译并且具备不同的`versionName`。
``` groovy
   productFlavors {
        beta {
            versionCode VERSION_CODE
            versionName VERSION_NAME + "_beta"
        }

        offical {
            versionCode VERSION_CODE
            versionName VERSION_NAME
        }
    }


    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

        beta {
            manifest.srcFile 'manifest/AndroidManifest_beta.xml'
        }

        offical {
            manifest.srcFile 'manifest/AndroidManifest_offical.xml'
        }
        instrumentTest.setRoot('tests')
    }
```

---
命令行编译
===
由于之前使用`Eclipse` Export 的包十之八九不能用，给我留下了心理阴影，所以我个人比较倾向于使用命令行编译，并且这种编译方式能更清晰的看到编译过程。以往若是要在命令行中使用`Ant`必须先 update一下项目，而`Android Studio`创建的项目同时会创建build.gradle，所以只要配置好环境变量，在项目根目录执行如下命令即可。
``` bash
$ gradle build
```

---
#### Reference
[Gradle Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide)