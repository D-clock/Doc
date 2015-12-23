Android-Gradle基础脚本解析
------

### Build.gradle

下面是一个Android module内的build.gradle脚本，Gradle是基于Groovy的一种DSL（领域特定语言）语言，在gradle里面，它每一对大括号称之为一个闭包
```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"

    defaultConfig {
        applicationId "com.clock.gradleusage"
        minSdkVersion 14
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        debug {

        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.3'
}
```
下面来分析这个脚本各个地方的意思和作用。

### Android Plugin
```groovy
apply plugin: 'com.android.application'
```
* 上面这段代码是告诉Gradle要加载一个`com.android.application`插件，在AndroidStudio下普通的module均做这样的声明
* 如果是library module的，则插件名变成`com.android.library`

### Manifest and defaultConfig

接下来这几个属相也相对比较熟悉，很多顾名思义，有些属性在原先的adt工程下是在Manifest文件中进行配置。
```groovy
	compileSdkVersion 21
    buildToolsVersion "21.1.2"

    defaultConfig {
        applicationId "com.clock.gradleusage"
        minSdkVersion 14
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }
```
* compileSdkVersion和buildToolsVersion：分别是编译SDK的版本号和构建这个module的工具的版本号
* applicationId: 当前module的包名
* 其他的属性都是之前Manifest里面有的，这里不赘述

### BuildTypes

这是一段构建类型的配置脚本
```groovy
	buildTypes {
        debug {

        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
```
* debug和release：分别是对打debug和release包的配置，AndroidStudio默认生成的构建脚本只有release的配置
* minifyEnabled： 是否进行代码混淆
* proguardFiles: 配置混淆规则，此处配置分为`proguard-android.txt`和`proguard-rules.pro`两部分，由这个两个文件共同完成代码混淆配置
* proguard-android.txt： AndroidStudio默认的混淆规则，在`AS目录\sdk\tools\proguard\proguard-android.txt`
* proguard-rules.pro: AndroidStudio为每个module生成的自定义混淆规则文件，在对应打包的module目录下

### LintOptions

这段是lint检查的配置，`checkReleaseBuilds false`和`abortOnError false`用来取消构建打包时候lint错误检查
```groovy
	lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
```

### Dependencies

最后这部分主要是对当前module依赖的配置
```groovy
	dependencies {
		compile fileTree(dir: 'libs', include: ['*.jar'])
		compile 'com.android.support:appcompat-v7:21.0.3'
	}
```
module下的依赖分为本地依赖和远程依赖两种。本地依赖我觉得也可以理解成为内部依赖，大概意思就是依赖的库是位于module之内，而远程依赖的话想对也比较好理解，就是所依赖的这个库在module之外，可以在Project下的其他地方，或者是在远程的服务器上
* `compile fileTree(dir: 'libs', include: ['*.jar'])`属于本地依赖，表示编译时会编译module下的libs文件夹中的所有jar文件，所以需要使用第三方jar的工程只需直接将jar加入到module下的libs文件夹即可
* `compile 'com.android.support:appcompat-v7:21.0.3'`属于远程依赖，表示依赖于module外的某个库
* 单独加入某个内部依赖也可以在上面的dependencies闭包中加入`compile files('libs/***.jar')`
* 引用了某个module的情况下也需要在dependencies闭包中加入`compile project(':引用依赖的module名')`

### 总结

抽空对gradle在Android开发下的应用做了个小小的总结，有一些功能还没提及到，如：`签名的配置`，`多渠道和自动化打包`，`生成jar或aar`等内容。还需要接下来再继续完善汇总。