# Gradle语法基础解析

在从ADT转移到AndroidStudio下开发，必然会遇到Gradle脚本打包的问题.看懂一个脚本最基本的前提就是了解它的语法，我在转移开发环境的过程中，也开始接触学习Gradle，在此做了一些总结，方便自己查阅.

## Gradle为何物

第一次接着Gradle，对它做了大致的了解。按照网上普遍的说法：**Gradle是以Groovy语言为基础，面向Java应用为主。基于DSL（领域特定语言）语法的自动化构建工具.** 看到这里我依旧还是有点云里雾里的，不过抓住了两个重点：
> 1.Gradle是一门语言
> 2.Gradle是一个自动化构建工具
既然单从概念上得不到很好的理解，那么作为学习一门语言和一个工具，只能通过使用来增强概念和功能上的了解了.

## Project和Task、Action

Gradle里面的任何东西都是基于**Project**和**Task**这两个概念，基于这两个概念，Gradle官方放出的指导手册是这么描述的：

* 每一个构建都是由一个或多个Project构成的.一个Project到底代表什么依赖于你想用Gradle做什么.举个例子,一个Project可以代表一个JAR或者一个网页应用.它也可能代表一个发布的 ZIP压缩包,这个ZIP可能是由许多其他项目的JARs构成的.但是一个Project不一定非要代表被构建的某个东西.它可以代表一件**要做的事,比如部署你的应用.
* 每一个Project是由一个或多个Task构成的.一个Task代表一些更加细化的构建.可能是编译一些classes,创建一个JAR,生成javadoc,或者生成某个目录的压缩文件.
* 每个Task又是由一个或多个Action构成的,Gradle中有两种类型的Action，分别是**doFirst**和**doLast**.

在AndroidStudio构建生成一个apk的安装包，它就要依赖于build.gradle脚本进行构建.此时**生成apk包**这样一件事情就可以理解成为一个Project（要做一件什么事），而生成apk包只是一个比较大一统的概念.打包的过程需要进行各种各样的配置，例如配置版本号，最低兼容Android几的平台，打包签名等.这些相当于**生成apk包**这个Project的一个个具体的子步骤，也就是Gradle中的Task.

## 基础语法

了解大概的一些基本概念之后，最重要的还是开始下手打码实战，创建自己的第一个Gradle构建脚本文件build.gradle

```groovy
task hello {
    doLast {
        println 'Hello world!'
    }
}
```

在命令行里,进入脚本所在的文件夹然后输入命令**gradle -q hello**来执行构建脚本（前提是你安装了Gradle并配置了环境变量），会在控制台窗口得到如下输出

```
$ gradle -q hello
Hello world!
```
这个命令所执行的事情可以分为以下几个步骤
1.去build.gradle文件中查找hello这个task，并且做编译执行；
2.执行hello task中每个action里面的流程，此处只有doLast{}一个action负责输出Hello world；

接下来看另外一段Gradle脚本

```groovy
task upper << {
	String someString = 'mY_nAmE'
	println "Original: " + someString
	println "Upper case: " + someString.toUpperCase()
}
```

执行**gradle -q upper**运行后，可以看到控制台窗口的输出如下：

```
$ gradle -q upper
Original: mY_nAmE
Upper case: MY_NAME
```

看到此处的代码，需要做一个简单的解释一下，上面的这段代码和下面的这种写法是等价的，上面的写法其实是Gradle提供的doLast{}的一种简写方式，因为Gradle直接重载了`<<`符号.

```groovy
task upper {
	doLast{
		String someString = 'mY_nAmE'
		println "Original: " + someString
		println "Upper case: " + someString.toUpperCase()
	}
}
```
看到这里，有没有发现其实Gradle的语法，其实跟Java是非常类似的,哈哈...其实网上也存在着一种说法：Groovy就是没有类型的Java,为什么说是Groovy，其实Gradle相当于Groovy的子类，Groovy的所有特性都被Gradle完整继承了.看完下面的代码就能理解为何成为没有类型的Java的原因了.

```groovy
task notype << {
	def oneInt = 1 //等价于 int oneInt = 1
	def oneFloat = 1.00 //等价于 ioneFloat = 1.00
	def oneString = 'Clock'//等价于 oneString = 'Clock'
	
	println "oneInt: " + oneInt
	println "oneFloat: " + oneFloat
	println "oneString: " + oneString
}
```

编译运行以上代码后，即可以看到以下输出

```
$ gradle -q notype
oneInt: 1
oneFloat: 1.00
oneString: Clock
```

之所以说Groovy是无类型的Java，就是因为不管所有的类型都可以使用**def（define）**来定义一个变量，Gradle会根据你赋值的类型，将变量转换成对应的基本类型.
最后来看一下Gradle里面如何使用循环的，直接看下面两段代码

```groovy
task rounder << {
	10.times{
		println "it is: " + it
	}
}
```

```groovy
task rounder << {
	10.times{a->
		println "it is: " + a
	}
}
```
上面的两段代码的执行结果相同，如下：
```
$ gradle -q rounder
it is: 0
it is: 1
it is: 2
it is: 3
it is: 4
it is: 5
it is: 6
it is: 7
it is: 8
it is: 9
```
都是做一个10次的循环，**.times** 和 **it**是关键字，其中**..times**.表示循环，**10.times**表示执行10次的一个循环，**it**表示循环中的计数值. 对于**it**，我们也可以自定义一个变量获取这个计数值，像第二段代码中的**a->**就是表示用**a**来取代**it**获取这个循环中的计数值.对于
```
println "it is: " + a

```
我们也可以等价写成
```
println "it is: $a"//$变量名，表示去取变量的值

```

## 任务依赖

Gradle中存在一种依赖关系，所谓依赖关系可以简单的描述成一个Task的执行需要已另一个Task作为基础，继续看下面的两段代码

```groovy
task hello << {
	println 'Hello world!'
}
task intro(dependsOn: hello) << {
	println "I'm Gradle"
}
```

```groovy
task intro(dependsOn: 'hello') << {
	println "I'm Gradle"
}
task hello << {
	println 'Hello world!'
}
```

上面两段代码的都是表示在执行intro task之前会先依赖执行hello task，**唯一的区别就是被依赖的task是定义在调用之前还是调用之后**，看到这里是否感觉这种依赖的关系相当于函数调用传入参数那样..显得非常易懂.

## 多项目和远程仓库

Gradle支持可以将一个Project划分成为一个或多个子Project来构成

```
include 'SubProject1','SubProject2','SubProject3'.........;
```
可以支持使用本地的mavenCentral库，或者是远程服务器上的库
```groovy
repositories {
    mavenCentral()//本地库支持
	maven {
        url "http://repo.mycompany.com/maven2" //远程库地址
    }

}
```

## 常用的Gradle命令

下面介绍一些Gradle中常用的命令操作
* 查看版本号: **gradle -v**
* 编译执行某个task: **gradle Task名**
* 静默编译执行某个task: **gradle -q Task名**（q表示quiet模式，表示编译执行Gradle脚本的过程中，只输出必要的信息. 除了quiet模式外，Gradle中还有其他三种模式）
* 编译执行某个Project中的某个task：**gradle -b Project名 Task名**（Gradle默认只执行build.gradle文件中，自定义其他文件xxx.gradle编译运行显式指定Project名称，这里的build.gradle其实表示的就是build Project）
* 显示所有的Project：**gradle projects**
* 显示所有的task：**gradle tasks**
* 显示gradle的gui：**gradle --gui** 或 **gradle --gui&**（后台运行）
* 查找所有的gradle命令: **gradle --help**

##最后

此处只是一小部分gradle的基础使用总结，更多的gradle使用方式请戳这里[Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html)