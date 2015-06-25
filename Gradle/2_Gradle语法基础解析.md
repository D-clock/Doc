# 2_Gradle语法基础解析

在从ADT转移到AndroidStudio下开发，必然会遇到Gradle脚本打包的问题.看懂一个脚本最基本的前提就是了解它的语法，我在转移开发环境的过程中，也开始接触学习Gradle，在此做了一些总结，方便自己查阅.

## Gradle为何物

第一次接着Gradle，对它做了大致的了解。按照网上普遍的说法：`Gradle是以Groovy语言为基础，面向Java应用为主。基于DSL（领域特定语言）语法的自动化构建工具.`看到这里我依旧还是有点云里雾里的，不过抓住了两个重点：
> 1.Gradle是一门语言
> 2.Gradle是一个自动化构建工具
既然单从概念上得不到很好的理解，那么作为学习一门语言和一个工具，只能通过使用来增强概念和功能上的了解了.

## Projects和tasks

Gradle里面的任何东西都是基于`Projects(项目)`和`Tasks(任务)`这两个概念，基于这两个概念，Gradle官方放出的指导手册是这么描述的：
* 每一个构建都是由一个或多个projects构成的.一个project到底代表什么依赖于你想用Gradle做什么.举个例子,一个project可以代表一个JAR或者一个网页应用.它也可能代表一个发布的 ZIP压缩包,这个ZIP可能是由许多其他项目的JARs构成的.但是一个project不一定非要代表被构建的某个东西.它可以代表一件**要做的事,比如部署你的应用.
* 每一个project是由一个或多个tasks构成的.一个task代表一些更加细化的构建.可能是编译一些classes,创建一个JAR,生成javadoc,或者生成某个目录的压缩文件.
在AndroidStudio构建生成一个apk的安装包，它就要依赖于build.gradle脚本进行构建.此时`生成apk包`这样一件事情就可以理解成为一个Project（要做一件什么事），而生成apk包只是一个比较大一统的概念.打包的过程需要进行各种各样的配置，例如配置版本号，最低兼容Android几的平台，打包签名等.这些相当于`生成apk包`这个Projects的一个个具体的子步骤，也就是Gradle中的Tasks.

## Hello world
了解大概的一些基本概念之后，最重要的还是开始下手打码实战，创建自己的第一个Gradle构建脚本build.gradle
```groovy
task hello {
    doLast {
        println 'Hello world!'
    }
}
```
在命令行里,进入脚本所在的文件夹然后输入`gradle -q hello`来执行构建脚本，会在控制台窗口得到如下输出
```
> gradle -q hello
Hello world!
```
这个整个build.gradle的构建脚本定义了一个独立的task，叫做hello，并且加入了一个action.当你运行`gradle hello`,Gradle执行叫做hello的task,也就是执行了你所提供的action.这个 action是一个包含了一些Groovy代码的闭包，上面叫做hello的task就是执行了一个doLast的action，这个action就是在控制台中输出`hello world`,gradle提供了doLast和doFirst这两种action.
从task的组成上看，其实task闭包的内部是被划分成为一个个的action组成.到此，我们也可以做一下如下关系数理：
> Project有一个或若干个Task组成，Task由一个或若干个Action组成
这里再补充说明一下`gradle -q`命令的作用，参数q表示Gradle构建的Quiet模式，用这种模式构建只会看到task中输出的信息，而构建过程中产生的其他信息一律不会在控制台的窗口中输出，
像平时在AS下编译android moudle时，你会看到Message窗口会输出类似下面这样的构建过程详细信息
```
Information:Gradle tasks [:baidumap:assembleDebug]
:baidumap:preBuild UP-TO-DATE
:baidumap:preDebugBuild UP-TO-DATE
:baidumap:checkDebugManifest
:baidumap:preReleaseBuild UP-TO-DATE
:baidumap:prepareComAndroidSupportAppcompatV72103Library UP-TO-DATE
:baidumap:prepareComAndroidSupportSupportV42103Library UP-TO-DATE
:baidumap:prepareDebugDependencies
:baidumap:compileDebugAidl UP-TO-DATE
:baidumap:compileDebugRenderscript UP-TO-DATE
:baidumap:generateDebugBuildConfig UP-TO-DATE
:baidumap:generateDebugAssets UP-TO-DATE
:baidumap:mergeDebugAssets UP-TO-DATE
:baidumap:generateDebugResValues UP-TO-DATE
:baidumap:generateDebugResources UP-TO-DATE
:baidumap:mergeDebugResources UP-TO-DATE
:baidumap:processDebugManifest UP-TO-DATE
:baidumap:processDebugResources UP-TO-DATE
:baidumap:generateDebugSources UP-TO-DATE
:baidumap:processDebugJavaRes UP-TO-DATE
:baidumap:compileDebugJava UP-TO-DATE
:baidumap:compileDebugNdk UP-TO-DATE
:baidumap:compileDebugSources UP-TO-DATE
:baidumap:preDexDebug UP-TO-DATE
:baidumap:dexDebug UP-TO-DATE
:baidumap:validateDebugSigning
:baidumap:packageDebug UP-TO-DATE
:baidumap:zipalignDebug UP-TO-DATE
:baidumap:assembleDebug UP-TO-DATE
Information:BUILD SUCCESSFUL
Information:Total time: 31.254 secs
Information:0 errors
Information:0 warnings
Information:See complete output in console
```
这种模式就是与Quiet模式恰好相反的，Gradle中出了Quiet模式外，还有其他几种构建模式，后面再做详细的学习总结。

## 