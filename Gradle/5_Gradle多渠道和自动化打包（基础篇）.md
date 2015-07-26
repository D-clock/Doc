# Gradle多渠道自动化打包（基础篇）

最近由于忙着公司项目上的开发和调整，一直没有多少时间停下来做笔记。今天继续记录一下AndroidStudio下如何使用Gradle打包的一些基本操作，同样希望能够帮到一些需要这些资源的开发者。代码上的编写非常简单，主要分为以下2步走即可

> **1.配置渠道；**
> 
> **2.配置渠道关键信息（如：设置渠道名，渠道ID）；**

## 配置渠道

说到配置渠道，本身来说非常简单，我们直接打开AndroidStudio为我们module生成的build.gradle构建脚本。在android的闭包里面添加productFlavors的闭包，并配置上我们所要打包的渠道

```groovy
android {
   ...
    productFlavors {
        wandoujia {}
        qihu360 {}
        baidu {}
    }
}
```

这里我添加了豌豆荚、360和百度手机助手作为我们app的渠道，到这里我们已经完成了第一步的配置。有木有so easy的赶脚。

## 配置渠道关键信息

接下来需要配置一些渠道关键信息，所谓的关键信息就是我们给渠道定义的**渠道号**（说白了就是对app分发渠道的一个标识，不懂的孩纸就需要自行上网搜索了）。渠道统计里面最经常使用的估计是非友盟莫属了。这里我们也以友盟作为例子

```xml
<meta-data
	android:name="UMENG_CHANNEL"
	android:value="origin" />
```

用过友盟的朋友都知道，我们需要在app的androidManifest.xml文件里面去配置这样一个渠道信息。以便友盟进行区别分析和统计。在平常开发过程中我们通常会写死在xml配置里面。而此时如果涉及到多渠道打包，我们就不能这么操作了。这里我们做一点小小的变化，把上面的值**origin**抽取到如下名为**CHANNEL_VALUE**的变量中

```xml
<meta-data
	android:name="UMENG_CHANNEL"
	android:value="${CHANNEL_VALUE}" />
```

到此，AndroidMainfest里面的活干完了，我们重新build.gradle的构建脚本里面去。在刚刚指定的渠道下面添加下面这样一段循环

```groovy
android {
   ...
    productFlavors {
        ...
    }
	productFlavors.all { flavor ->
        flavor.manifestPlaceholders = [CHANNEL_VALUE: name]
    }
}
```

为我们生成每个渠道包的AndroidManifest里面的变量**CHANNEL_VALUE**赋值。到此，我们就能顺利的用友盟对我们的渠道包进行区别统计了。

## 打包

完成了上面的配置之后，我们就可以在AndroidStudio进行批量的打包了。

1.点击生成签名包

![点击生成签名包](http://c.hiphotos.baidu.com/image/pic/item/8b82b9014a90f603e122c4473f12b31bb051ed24.jpg)

2.勾选要打包的moudle

![勾选要打包的moudle](http://b.hiphotos.baidu.com/image/pic/item/5243fbf2b21193134174a4b063380cd791238dff.jpg)

3.选择打包的数字签名(如果没有就自己点击create new...创建一个)

![选择打包的数字签名](http://f.hiphotos.baidu.com/image/pic/item/91529822720e0cf31a6516fb0c46f21fbe09aa24.jpg)

4.选择好生成包的路径和渠道包名(可以看到我们配置好的渠道都在里面)

![选择好生成包的路径和渠道包名](http://d.hiphotos.baidu.com/image/pic/item/d31b0ef41bd5ad6ebe233cea87cb39dbb7fd3cd3.jpg)

5.点击finish静静的等待生成渠道包就可以了。

![生成渠道包](http://b.hiphotos.baidu.com/image/pic/item/a50f4bfbfbedab64823815c0f136afc379311e63.jpg)

## 验证

为了更好的看到打包是否生效，我直接拿了生成的baidu渠道包**app-baidu-release.apk**来做一个反编译(对反编译工具有兴趣的朋友请戳这里[Android反编译工具系列](/Tools/Android反编译工具系列.md))的验证

>反编译xml文件

![反编译xml文件](http://d.hiphotos.baidu.com/image/pic/item/3812b31bb051f81915df444adcb44aed2e73e73c.jpg)

>反编译成功得到的文件
>
![反编译成功得到的文件](http://b.hiphotos.baidu.com/image/pic/item/9f2f070828381f30487fcba9af014c086e06f048.jpg)

>打开AndroidMainfest验证结果

![打开AndroidMainfest验证结果](http://b.hiphotos.baidu.com/image/pic/item/6a63f6246b600c33568399c81c4c510fd9f9a13c.jpg)

经过反编译后，最终可以确定多渠道打包成功。如果想知道更多gradle自动化多渠道打包的知识，欢迎继续阅读下一篇笔记[Gradle多渠道和自动化打包（深入篇）](6_Gradle多渠道和自动化打包（深入篇）.md)

最后，献上完整的基础打包代码

```groovy
apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"

    defaultConfig {
        applicationId "com.clock.gradle"
        minSdkVersion 14
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    productFlavors {
        wandoujia {}
        qihu360 {}
        baidu {}
    }

    productFlavors.all { flavor ->
        flavor.manifestPlaceholders = [CHANNEL_VALUE: name]
    }

}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.3'
}

```