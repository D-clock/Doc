# ProGuard基础语法和打包配置(AndroidStudio打包补充)

继前一篇笔记[3_1_ProGuard基础语法和打包配置](3_1_ProGuard基础语法和打包配置.md)总结了一些打包混淆需要注意的地方。这几天忙着新项目的上线，切换到AndroidStudio下开发后，需要写一下gradle的打包脚本，因为是新项目，也需要重新写一下混淆配置文件。结果碰到了下面一个问题**混淆打包的时候移除打log的代码的配置不成功**。前一篇笔记说过，打包的时候做了如下的配置就可以移除工程里面的log，避免打包发布后的一些问题.

```proguard
-assumenosideeffects class android.util.Log{
    public static int v(...);
    public static int i(...);
    public static int d(...);
    public static int w(...);
    public static int e(...);
}

-assumenosideeffects class java.io.PrintStream{
    public void println(%);
    public void println(**);
}
```

结果在自定义的混淆文件下，捣腾了很久发现一点效果都木有，郁闷了很久，莫非**ProGuard的这些处理已经失效了？**脑子里满是大问号，Google了很久也没有发现什么线索，索性就先丢了一边。但是作为一个强迫症程序员，过几天又想起了这个问题，有空便索性研究琢磨一下，还真的找到问题的所在，顺利的解决了。

## 问题出在哪？

思前想后，发现Google了很久都没有所以然，有溜达去ProGuard的官网看了一下语法，也没有提及废弃到上面那种优化方式。那么，基本可以断定是我自己的配置出了问题。但我仔细检查了自己的配置，发现并无异样，网上提及到的这种优化方式的失效的情况，也只有在混淆文件配置了

* -dontoptimize（顾名思义，don't optimize 不优化）

的时候才回导致失效，可是问题我压根自己的配置文件并没有配这个啊啊啊啊啊啊啊！额，脑子一转，不对，我用gradle来打包的，AndroidStudio生成build.gradle打包文件的时候，就指定了一个默认的的混淆文件**proguard-android.txt**（在 SDK目录/tools/proguard/ 目录下面）另外一个让我们自行添加混淆规则的**proguard-rules.pro**文件就在我们写代码的moudle目录下面。

```groovy
buildTypes {
	release {
		minifyEnabled true
		proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
	}
}
```

如此一来，自己写的**proguard-rules.pro**里面没问题，那么基本是在**proguard-android.txt**里面出了问题，索性直接打开那个文件看看，果不其然，看到了下面的两行混淆配置

```proguard
-dontoptimize
-dontpreverify
```

这样就能明白自己去除log的代码失效的原因了。哈哈，找到罪魁祸首后，直接把这个配置去掉就ok了，鸡冻得赶紧重新打包享受胜利的果实。结果打包到了一半，妈蛋，直接报错终止了。

## 去了-dontoptimize还不行

额，开了好久，发现被打回原点。no，还达不到原点，连打包都打不了。是不是还要什么地方需要改动呢，后来当天没想到什么好方法，索性隔了一天再回来捣腾，又发现了一些问题，在sdk自带的配置混淆文件的目录下，有着三个混淆的配置文件

* proguard-android.txt
* proguard-android-optimize.txt
* proguard-project.txt

打开之前的**proguard-android.txt**看到了下面这段注释

```
-dontoptimize
-dontpreverify
# Note that if you want to enable optimization, you cannot just
# include optimization flags in your own project configuration file;
# instead you will need to point to the
# "proguard-android-optimize.txt" file instead of this one from your
# project.properties file.
```

哈哈，这回真是如梦初醒啊啊啊啊啊啊！，赶紧把gradle里面的系统混淆配置文件改成**proguard-android-optimize.txt**，再搭上自己写的那份混淆文件一起打包，过了一会搞定，终于打出一个我想要的混淆包出来了

## 总结

仔细的去看了一下**proguard-android.txt**和**proguard-android-optimize.txt**的区别，其实主要的问题是出现在了少了下面的这段配置

```
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
-optimizationpasses 5
-allowaccessmodification
-dontpreverify

# The remainder of this file is identical to the non-optimized version
# of the Proguard configuration file (except that the other file has
# flags to turn off optimization).

-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-verbose
```

打包加入优化需要指定优化等级和优化策略算法的一些配置等等，因为去了-dontoptimize后没有配置上面这些，所导致连包都打不了了。

另外，**instead you will need to point to the "proguard-android-optimize.txt" file instead of this one from your project.properties file.**这提示其实是旧的开发工具ADT下面的提示而已，因为ADT下混淆配置是在**project.properties**指定。换成AndroidStudio开发的话，去build.gradle的构建脚本中配置就好了，如下

```groovy
buildTypes {
	release {
		minifyEnabled true
		proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
	}
}
```

啰里吧嗦又一篇，与君分享，不喜勿喷。。哈哈