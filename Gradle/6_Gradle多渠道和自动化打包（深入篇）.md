# Gradle多渠道和自动化打包（深入篇）

继[Gradle多渠道和自动化打包（基础篇）](5_Gradle多渠道和自动化打包（基础篇）.md)后，接着记录一下多渠道自动化打包的另外一些配置操作，主要分为以下4个方面

> **1.一个渠道多个信息**

> **2.打包签名配置**

> **3.修改生成apk包名**

> **4.设置编译时的渠道信息**

> **5.突破65535方法的限制**


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

![创建配置文件](http://g.hiphotos.baidu.com/image/pic/item/ca1349540923dd5470c3e469d709b3de9c82487a.jpg)

在这里面通过键值对的方式写好签名的关键信息

![填写签名信息](http://a.hiphotos.baidu.com/image/pic/item/37d12f2eb9389b50473da3538335e5dde6116ef4.jpg)

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




## 最后


![整个打包流程图](http://a.hiphotos.baidu.com/image/pic/item/a71ea8d3fd1f4134af87467f231f95cad1c85e7a.jpg)


更多关于Gradle打包的详情，移步到官方的文档（**需要翻墙**）[官方原文请戳这里](http://tools.android.com/tech-docs/new-build-system/user-guide)

也可以直接在Github上看看随手记的开发者翻译出来的文档 [中文译本请戳这里](https://github.com/rujews/android-tech-docs/blob/master/new-build-system/user-guide/README.md)