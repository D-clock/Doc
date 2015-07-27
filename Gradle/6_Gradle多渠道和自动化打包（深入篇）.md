# Gradle多渠道和自动化打包（深入篇）

继[Gradle多渠道和自动化打包（基础篇）](5_Gradle多渠道和自动化打包（基础篇）.md)后，接着记录一下多渠道自动化打包的另外一些配置操作，主要分为以下5个方面

> **1.一个渠道多个信息**

> **2.打包签名配置**

> **3.修改生成apk包名**

> **4.设置编译时的渠道信息**

> **5.其他**


## 添加多个渠道信息

上一面文章里面给出是示例，只是简单的给**UMENG_CHANNEL**打上不同的渠道名。那么，如果我想要为每个渠道名添加一个对于的渠道ID，那应该怎么做咧？首先，我在原先生成友盟渠道名的meta-data那里创建一个新的meta-data，作为标识每个渠道的ID。

```xml
<meta-data
	android:name="UMENG_CHANNEL"
    android:value="${CHANNEL_VALUE}" />

<meta-data
    android:name="CHANNEL_ID"
    android:value="${CHANNEL_ID}" />

```

按照我的想法，豌豆荚，360手机助手和百度手机助手的渠道id分别设置成200、201、202。于是，移出原先的循环，只能直接去到各个渠道的闭包下做配置。

```groovy

android {
    ...

    productFlavors {
        wandoujia {
            manifestPlaceholders = [CHANNEL_VALUE: 'wandoujia' , CHANNEL_ID:200]
        }
        qihu360 {
            manifestPlaceholders = [CHANNEL_VALUE: '360' , CHANNEL_ID:201]
        }
        baidu {
            manifestPlaceholders = [CHANNEL_VALUE: 'baidu' , CHANNEL_ID:202]
        }
    }

}

```

到这里就可以在打包的时候成功打上渠道名和其对应的ID号了。如果想要验证结果的话，还是可以去反编译查看一下。

## 打包签名配置

打包签名的配置分为两种情况，一种是采用AndroidStudio来打包，这样只需要在步骤中勾选好打包的签名文件已经填写相关的密码信息，即可完成；另外一种，是进入到命令行界面进行打包操作，这时候就需要我们在build.gradle脚本中做好如下的签名信息配置

```groovy

android {
    ...

    signingConfigs {
        debug {

        }
        release {
            storeFile '打包签名路径'
            storePassword 'XXX'
            keyAlias 'XXX'
            keyPassword 'XXX'
        }
    }

}

```

但这种做法是**非常不建议**的，因为这样会导致我们签名信息泄露，且不方便于我们进行控制和管理。所以，官方提倡的做法是，把这些关键信息存放到文件中去，然后再gradle打包脚本中读取信息。所以，我直接在当前的module下面创建了一个**signing.properties**的配置文件，

![创建配置文件](http://e.hiphotos.baidu.com/image/pic/item/e850352ac65c103897436cc3b4119313b07e89f6.jpg)

在这里面通过键值对的方式写好签名的关键信息

![填写签名信息](http://a.hiphotos.baidu.com/image/pic/item/7c1ed21b0ef41bd595404af557da81cb39db3d9b.jpg)

再根据下面的gradle打包代码，从这份文件中读取签名信息出来打包

```groovy

android {
    ...

    signingConfigs {
        debug {

        }
        release {
            storeFile 
            storePassword 
            keyAlias 
            keyPassword 
        }
    }

	buildTypes {
        release {
            signingConfig signingConfigs.release //设置签名信息
        }
    }

}

//读取文件中的信息来配置打包签名
File propFile = file('signing.properties');
if (propFile.exists()) {
    def Properties props = new Properties()
    props.load(new FileInputStream(propFile))
    if (props.containsKey('STORE_FILE') && props.containsKey('STORE_PASSWORD') &&
            props.containsKey('KEY_ALIAS') && props.containsKey('KEY_PASSWORD')) {
        android.signingConfigs.release.storeFile = file(props['STORE_FILE'])
        android.signingConfigs.release.storePassword = props['STORE_PASSWORD']
        android.signingConfigs.release.keyAlias = props['KEY_ALIAS']
        android.signingConfigs.release.keyPassword = props['KEY_PASSWORD']
    } else {
        android.buildTypes.release.signingConfig = null
    }
} else {
    android.buildTypes.release.signingConfig = null
}

	....

```

TeamLeader可以在提交代码的时候把**signing.properties**文件忽略掉，这样就不会将签名等信息泄露出去，对app的安全造成威胁。同时，也方便TeamLeader进行控制管理。

## 修改生成apk包名

在使用AndroidStudio打渠道包的时候，生成的渠道包默认都是以**app名-渠道名-release.apk**的格式命名。如果你想在让生成的渠道包进行自定义的格式命名，那么可以在gradle脚本中这样做

```groovy

apply plugin: 'com.android.application'

//获取产品的名字
def getProductName() {
    return "clock"
}
//获取当前系统的时间
def releaseTime() {
    return new Date().format("yyyyMMdd", TimeZone.getTimeZone("UTC"))
}

android {
    ...

    signingConfigs {
        ...
    }

	buildTypes {
        ...
    }
	//修改生成的apk名字，格式为 app名_版本号_打包时间_渠道名_release.apk
    applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def oldFile = output.outputFile
            if (variant.buildType.name.equals('release')) {
                def releaseApkName = getProductName() + "_v${defaultConfig.versionName}_${releaseTime()}_" + variant.productFlavors[0].name + '_release.apk'
                output.outputFile = new File(oldFile.parent, releaseApkName)
            }
        }
    }

}
	....

```

按照上面的代码进行打包之后，生成的包名就会变成我们重新指定的格式。这样肯定比我们手工操作方便多了，码农的任务就是要用代码解放双手，哈哈。

## 设置编译时的渠道信息

在我们配置好打发布包的脚本时，随之也会遇到一个问题。就是，我们再AndroidManifest里面做的配置

![AndroidManifest配置](http://a.hiphotos.baidu.com/image/pic/item/0e2442a7d933c89599d4317ad71373f08202000c.jpg)

在平常的开发调试模式下，这两个值它会变成什么值呢？是不是突然有点懵了...别懵，这时候你只要来到androidStudio的左下角点开**Build Variants**窗口

![Build Variants](http://c.hiphotos.baidu.com/image/pic/item/aa64034f78f0f736fd46e6000c55b319ebc41372.jpg)

点击下拉，就可以勾选我们开发模式下要生成什么渠道信息了。

![选择调试模式渠道](http://a.hiphotos.baidu.com/image/pic/item/622762d0f703918f28ada8cb573d269759eec4b7.jpg)

如果你的开发工具左下角没有**Build Variants**窗口，那自己去到**工具栏>View>Tool Windows**里面打开。

## 其他

除了以上的一些配置外，在打包的时候还有另外一些配置问题，如：配置混淆规则，配置混淆优化，移除无用的资源文件，压缩对齐apk包，突破65535个方法限制等等。这里主要记录一下最后的65535方法限制的问题，其他的会在最后给出完整的打包脚本。

Android生成的app的apk包要求所有方法的总数必须小于65536个方法，否则就无法生成单个class.dex文件（干过反编译这事的人应该都熟悉这东东）。所以，一个app在不断发展迭代的过程之后，必然会面临这个问题。gradle给出的解决方法很简单


```groovy

android {
    ...
    buildTypes {
        release {
            ...
			multiDexEnabled true//启用生成个dex文件的支持
        }
    }


dependencies {
    compile fileTree(dir: 'libs', include: '*.jar')
    compile files('libs/locSDK_5.01.jar')
    compile 'com.umeng.analytics:analytics:latest.integration'
	//添加支持multidex的兼容包
    compile 'com.android.support:multidex:1.0.0'
}

```

除此之外，我们还需要继承MultiDexApplication，并重写attachBaseContext方法

```java

/**
 * 处理解决mutiDex的问题
 *
 * Created by Clock on 2015/7/23.
 */
public class BaseApplication extends MultiDexApplication{

    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
		//支持多dex的apk安装
        MultiDex.install(this);
    }
}

```

这样就可以解决问题了，不过因为Android系统的各种原因，这种方式只能够在Android4.0开始及之后的平台生效。因为在Multidex在2.x的系统上因为突然需要开辟一大块内存，所以会造成崩溃而无法使用的问题。


## 最后

好了，本次的笔记就到此了。记录能够帮助自己消化知识，同时也分享给其他需要的开发者。最后献上一份完整的build.gradle打包脚本，还有一张打包生成apk的整个流程图。

```groovy
apply plugin: 'com.android.application'

//获取生成的产品名
def getProductName() {
    return "clock"
}

//获取打包的时间
def releaseTime() {
    return new Date().format("yyyyMMdd", TimeZone.getTimeZone("UTC"))
}


android {
    compileSdkVersion 21
    buildToolsVersion "21.1.2"

    packagingOptions {
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
        exclude 'META-INF/MANIFEST.MF'
    }

    lintOptions {
		//忽略一些构建信息，降低对错误的检查
        checkReleaseBuilds false
        abortOnError false
    }

    defaultConfig {
        applicationId "com.clock.chatlink"
        minSdkVersion 9
        targetSdkVersion 21
        versionCode 1
        versionName "1.0"
    }

    signingConfigs {
        debug {

        }
		//配置签名的关键信息
        release {
            storeFile
            storePassword
            keyAlias
            keyPassword
        }
    }

    buildTypes {
        release {
			//启用混淆代码的功能
            minifyEnabled true
			//压缩对齐生成的apk包
            zipAlignEnabled true
			//指定混淆规则，需要压缩优化的混淆要把proguard-android.txt换成proguard-android.txt
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
			//指定打release包的签名
            signingConfig signingConfigs.release
			//移除无用的资源文件
            shrinkResources true
            //启用multidex的支持
            multiDexEnabled true
        }
    }

    productFlavors {
        wandoujia {
            manifestPlaceholders = [CHANNEL_VALUE: 'wandoujia' , CHANNEL_ID:200]
        }
        qihu360 {
            manifestPlaceholders = [CHANNEL_VALUE: '360' , CHANNEL_ID:201]
        }
        baidu {
            manifestPlaceholders = [CHANNEL_VALUE: 'baidu' , CHANNEL_ID:202]
        }
    }

    //修改生成的apk名字
    applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def oldFile = output.outputFile
            if (variant.buildType.name.equals('release')) {
                def releaseApkName = getProductName() + "_v${defaultConfig.versionName}_${releaseTime()}_" + variant.productFlavors[0].name + '_release.apk'
                output.outputFile = new File(oldFile.parent, releaseApkName)
            }
        }
    }
}

//配置打包签名
File propFile = file('signing.properties');
if (propFile.exists()) {
    def Properties props = new Properties()
    props.load(new FileInputStream(propFile))
    if (props.containsKey('STORE_FILE') && props.containsKey('STORE_PASSWORD') &&
            props.containsKey('KEY_ALIAS') && props.containsKey('KEY_PASSWORD')) {
        android.signingConfigs.release.storeFile = file(props['STORE_FILE'])
        android.signingConfigs.release.storePassword = props['STORE_PASSWORD']
        android.signingConfigs.release.keyAlias = props['KEY_ALIAS']
        android.signingConfigs.release.keyPassword = props['KEY_PASSWORD']
    } else {
        android.buildTypes.release.signingConfig = null
    }
} else {
    android.buildTypes.release.signingConfig = null
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.3'
    compile 'com.umeng.analytics:analytics:latest.integration'
	//添加multidex的外部依赖jar包
    compile 'com.android.support:multidex:1.0.0'
}


```

下面给的整个打包流程图，可以让大家清晰的看到整个打包流程都做了些什么。

![整个打包流程图](http://a.hiphotos.baidu.com/image/pic/item/a71ea8d3fd1f4134af87467f231f95cad1c85e7a.jpg)


更多关于Gradle打包的详情，移步到官方的文档（**需要翻墙**）[官方原文请戳这里](http://tools.android.com/tech-docs/new-build-system/user-guide)

也可以直接在Github上看看随手记的开发者翻译出来的文档 [中文译本请戳这里](https://github.com/rujews/android-tech-docs/blob/master/new-build-system/user-guide/README.md)

> **后续会再记录更多关于gradle使用的笔记，有兴趣的朋友敬请期待...**