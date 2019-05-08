# Gradle配置简单使用说明
使用Android Studio已经有好长一段时间了，老实说，1.x版本的时候，还是蛮蛋疼的，经常导入一个库不是这里错就是那里错有时候半天都跑不起来，最后发现大多数时候都是由于自己对Gradle不太熟悉导致的，那么这里记录下自己对Gradle配置使用的一些心得。

## Android Studio中各个Gradle File
#### Gradle File
首先我们一个最简单的项目中Gradle File的结构是这样的

 ![](imges/gradle.png)

#### setting.gradle
其中project中的setting.gradle是最简单的一个了，它规定了当前project中到底有多少个Module参与编译,如下就表示当前项目中的app、mylibrary两个module参与编译

```groovy
include ':app', ':mylibrary'
```

#### Project 中的build.gradle
- 默认的build.gradle

PS:这个文件中的配置会作用于整个工程，也就是所有参与编译的module，这也就决定了这个配置文件当然使用于定义一些共用的东西，我们先看下最简单的
```groovy
buildscript {

    //buildscript是针对gradle脚本本身执行所需的配置
    
    repositories {
        //这里定义的是gradle脚本本身执行所需的依赖查找路径
        google()
        jcenter()
    }
    
    dependencies {
        //此处定义编译插件的版本，这里就决定了我们编译过程中都会有哪些task（当然，并不包括我们自己定义的task）
        classpath 'com.android.tools.build:gradle:3.1.4'
    }
}

allprojects {

    //针对整个项目编译所需的配置
    
    repositories {
        //这里定义的使项目本身所需的依赖的查找路径，当然，这里我们可以指定多个。。。
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```
- 自定义全局通用属性

假如我们在整个项目的所有module中的build.gradle都用到的属性，那么可以考虑定义在project的build.gradle文件中,比如说如下字段
```groovy
ext{
    compileSdkVersion = 28
    buildToolsVersion = 28
    minSdkVersion = 15
    targetSdkVersion = 28
    versionCode = 1
    versionName = "1.0"
}
```

#### module中的build.gradle