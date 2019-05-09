# Gradle配置简单使用说明
使用Android Studio已经有好长一段时间了，老实说，1.x版本的时候，还是蛮蛋疼的，经常导入一个库不是这里错就是那里错有时候半天都跑不起来，最后发现大多数时候都是由于自己对Gradle不太熟悉导致的，那么这里记录下自己对Gradle配置使用的一些心得。

## Android Studio中各个Gradle File
### Gradle File
首先我们一个最简单的项目中Gradle File的结构是这样的

 ![](imges/gradle.png)

### setting.gradle
其中project中的setting.gradle是最简单的一个了，它规定了当前project中到底有多少个Module参与编译,如下就表示当前项目中的app、mylibrary两个module参与编译

```groovy
include ':app', ':mylibrary'
```

### Project 中的build.gradle
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
    buildToolsVersion = "28.0.0"
    minSdkVersion = 15
    targetSdkVersion = 28
    versionCode = 1
    versionName = "1.0"
}
```

### module中的build.gradle
- 首先我们看下最简单的模式
```groovy
//决定编译目标,com.android.application表示编译成android应用，com.android.library表示编译成依赖库，，，，
apply plugin: 'com.android.application'

android {
    //编译规范SDK版本（在编译的时候起到代码检查和警告作用，有利于检查一些第三方库，或者代码的兼容性）
    compileSdkVersion rootProject.compileSdkVersion
    //编译工具版本(用于规范所有开发人员的编译版本)
    buildToolsVersion rootProject.buildToolsVersion

    defaultConfig {
        applicationId "com.discovery.gradleoverview"
        //应用支持的最低系统版本
        minSdkVersion rootProject.minSdkVersion
        //目标软件开发版本（就是我们应用程序开发过程中所使用的API版本，这个非常重要，关系到后续我们的程序运行行为）
        targetSdkVersion rootProject.targetSdkVersion
        //内部版本号，多数用于更新
        versionCode rootProject.versionCode
        //外部版本号，用于显示给用户看的
        versionName rootProject.versionName
        //用于支持单元测试
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

//依赖库
dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
    implementation 'com.google.code.gson:gson:2.8.5'
    implementation project(':module_common')
    implementation files('libs/log4j-core-2.3.jar')
}
```
我们看到除了apply plugin之外，build.gradle主要分为android和dependencies，那么我们接下来围绕这两个展开

- dependencies
dependencies节点是管理我们添加的依赖，分别有如下几种方式添加依赖：
1、implementation files('libs/log4j-core-2.3.jar')
2、implementation project(':module_common')
3、implementation 'com.google.code.gson:gson:2.8.5'

其中第一第二种就不说了，添加的使本地依赖，第三种就比较特别了，是在线获取的，其实最后还是引入jar或者使arr，就是上面的gson一样最后同步完成后我们使可以在工程根目录的External Libraries节点下找到相关的jar包的，如下图：
![](images/gson.png)
那么这个jar是在哪来的呢？不知道大家还记不记得我们前面说的project build.gradle文件下有一个allprojects节点，里面的repositories中指定了若干代码仓库的，这个jar就是来自这里的其中一个仓库了。。。
