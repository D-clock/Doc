# AndroidStudio插件系列

终于有空来整理分享一下一些不错的AS插件了~~~

## Genymotion

Genymotion是测试Android应用程序，使你能够运行Android定制版本的旗舰工具。它是为了VirtualBox内部的执行而创建的，并配备了一整套与虚拟Android环境交互所需的传感器和功能。使用Genymotion能让你在多种虚拟开发设备上测试Android应用程序，并且它的模拟器比默认模拟器要快很多。

https://www.genymotion.com/

![Genymotion](http://g.hiphotos.baidu.com/image/pic/item/d009b3de9c82d15832cbc823860a19d8bd3e42d3.jpg)

## Android ButterKnife Zelezny

Android ButterKnife是一个“Android视图注入库”。它提供了一个更好的代码视图，使之更具可读性。 ButterKnife能让你专注于逻辑，而不是胶合代码用于查找视图或增加侦听器。用ButterKnife编程，你必须对任意对象进行注入，注入形式是这样的：

```java
	@InjectView(R.id.title) TextView title;
```

![ButterKnife](http://b.hiphotos.baidu.com/image/pic/item/30adcbef76094b3665c3887fa5cc7cd98c109dcb.jpg)

https://github.com/avast/android-butterknife-zelezny

> **一般情况下还是不建议用太多的注解**

## Robotium Recorder

Robotium Recorder是一个自动化测试框架，用于测试在模拟器和Android设备上原生的和混合的移动应用程序。Robotium Recorder可以让你记录测试案例和用户操作。你也可以查看不同Android活动时的系统功能和用户测试场景。

Robotium Recorder能让你看到当你的应用程序运行在设备上时，它是否能按预期工作，或者是否能对用户动作做出正确的回应。如果你想要开发稳定的Android应用程序，那么此插件对于进行彻底的测试很有帮助。

![Robotium Recorder](http://b.hiphotos.baidu.com/image/pic/item/03087bf40ad162d98b54f18e17dfa9ec8b13cdee.jpg)

http://robotium.com/products/robotium-recorder

> **要钱的哈，天朝找找破解**

## SelectorChapek

设计师给我们提供好了各种资源，每个按钮都要写一个selector是不是很麻烦？这么这个插件就为解决这个问题而生，你只需要做的是告诉设计师们按照规范命名就好了，其他一键搞定。

![SelectorChapek](http://e.hiphotos.baidu.com/image/pic/item/1ad5ad6eddc451daadd39f79b0fd5266d016327e.jpg)

https://github.com/inmite/android-selector-chapek

## GsonFormat

现在大多数服务端api都以json数据格式返回，而客户端需要根据api接口生成相应的实体类，这个插件把这个过程自动化了，赶紧使用起来吧

![GsonFormat](http://f.hiphotos.baidu.com/image/pic/item/f7246b600c33874476a1bdee570fd9f9d72aa038.jpg)

https://github.com/zzz40500/GsonFormat

## ParcelableGenerator

Android中的序列化有两种方式，分别是实现Serializable接口和Parcelable接口，但在Android中是推荐使用Parcelable，只不过我们这种方式要比Serializable方式要繁琐，那么有了这个插件一切就ok了。

![ParcelableGenerator](http://c.hiphotos.baidu.com/image/pic/item/a08b87d6277f9e2f6d628f9a1930e924b999f3c9.jpg)

https://github.com/mcharmas/android-parcelable-intellij-plugin

## SQLScout

一个黑科技的AS插件，有多黑，自己进主页围观膜拜吧！需要付费，有一个月的免费使用时间。天朝子民可以自己找破解方式了。

http://www.idescout.com/


## 总结

好了，大致就先到这里了，有nice的插件会继续更新...